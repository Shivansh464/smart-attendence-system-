# Smart Attendance Management System

A production-oriented, offline-first Android app that lets a professor take classroom
attendance by walking students past the phone's camera. Faces are detected, aligned, embedded,
and matched entirely on-device using **InsightFace** models (SCRFD detector + ArcFace
embedder) running via ONNX Runtime Mobile -- no student ever installs anything or logs in.

```
Professor logs in -> picks Department/Semester/Section/Subject -> taps Start Attendance
  -> each student's face is detected, aligned, embedded, and matched against the enrolled
     roster -> a match marks PRESENT once per lecture -> Stop -> synced to the backend
     whenever internet is available.
```

## Repository layout

```
SmartAttendanceSystem/
├── android-app/     Kotlin + Jetpack Compose + MVVM app (Hilt, Room, CameraX, ONNX Runtime)
├── backend/         Kotlin + Spring Boot REST API (JWT auth, PostgreSQL via Flyway)
├── ml/              Script to fetch/export the InsightFace ONNX models into the app's assets
└── docs/            Architecture, schema, model setup, install/deploy/testing guides
```

## Start here

| Doc | What's in it |
|---|---|
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | System diagram, the full detect->align->embed->match pipeline, why offline-first + REST (not Firestore) |
| [`docs/DATABASE_SCHEMA.md`](docs/DATABASE_SCHEMA.md) | ER diagram, table-by-table rationale, how duplicate-attendance prevention actually works |
| [`docs/MODEL_SETUP.md`](docs/MODEL_SETUP.md) | **Do this first** -- fetching the InsightFace ONNX models, no app screen works without it |
| [`docs/INSTALLATION.md`](docs/INSTALLATION.md) | Local dev setup, step by step, ending in a first-run smoke test |
| [`docs/DEPLOYMENT_GUIDE.md`](docs/DEPLOYMENT_GUIDE.md) | Production hosting, release signing, scaling to QR/fingerprint/liveness later |
| [`docs/TESTING_PLAN.md`](docs/TESTING_PLAN.md) | What's already automated vs. what needs a real device + real faces |

## What's implemented

- **Face recognition**: SCRFD detection + landmark alignment + ArcFace embedding, all on-device
  ([`face/`](android-app/app/src/main/java/com/smartattendance/app/face)); cosine-similarity
  matching against every enrolled student's embedding(s), threshold-gated.
- **Attendance workflow**: CameraX live capture, per-lecture duplicate prevention
  (`UNIQUE(session_id, student_id)` at both the Room and Postgres layers), live present-count UI,
  attendance-percentage recalculation on lecture close.
- **Professor-only auth**: JWT issued by the Spring Boot backend, stored in
  `EncryptedSharedPreferences` on-device; there is no student login anywhere in the system.
- **Offline-first sync**: every write lands in Room first; `SyncWorker` (WorkManager, network-
  constrained) pushes to the backend idempotently via client-generated ids, so a retried sync
  after a dropped connection never duplicates data.
- **Low-attendance alerts**: sub-40% students surface on the Analytics screen with a prefilled
  WhatsApp intent to the parent's number (no Meta Business API dependency for the MVP path).
- **Reports**: PDF (`android.graphics.pdf.PdfDocument`) and Excel (Apache POI) export, shared via
  a `FileProvider`.
- **Backend**: Spring Boot + PostgreSQL (Flyway-migrated schema), JWT security filter, idempotent
  sync endpoint, read-only reporting endpoints.

## What's intentionally left for you to finish

- **Model weights**: not committed (large binaries with their own license terms) -- see
  `docs/MODEL_SETUP.md`. The app fails fast with a clear error until this step is done.
- **SCRFD output tensor names**: `FaceDetector.kt` matches the standard InsightFace export
  naming; verify against your specific exported model in Netron once (documented in
  `docs/MODEL_SETUP.md`).
- **Release signing config** for the Android app, and a production `JWT_SECRET` /
  `API_BASE_URL` -- see `docs/DEPLOYMENT_GUIDE.md`.
- **WhatsApp Business API**: the MVP alert path is a prefilled `wa.me` intent the professor taps
  to send (no external account needed). Swapping in the official Business API later is a
  single-seam change at `alerts/WhatsAppAlertHelper.kt`.

## License note

This project integrates model weights from the InsightFace model zoo, which are released for
non-commercial research use under their own terms -- see `docs/MODEL_SETUP.md`'s licensing
section before any commercial deployment.
