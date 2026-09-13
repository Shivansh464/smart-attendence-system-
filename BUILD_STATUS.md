# APK build status

Building the debug APK using the toolchain already installed on this machine:
- JDK: `D:\jdk-21` (OpenJDK 21.0.11)
- Android SDK: `D:\android-sdk` (platforms 34/35/36, build-tools 36.0.0)
- Gradle: `9.1.0` (found cached under `~/.gradle/wrapper/dists`)

Command running: `gradle assembleDebug` from `android-app/`

**Note:** the app will build and install fine, but face recognition will not work at runtime
until the InsightFace ONNX model files are added to
`android-app/app/src/main/assets/models/` (see `docs/MODEL_SETUP.md`) -- they're not committed
because they're large binaries with their own license terms. Login, Dashboard, Students list,
History, Analytics, and Export all work without them; only the camera recognition screen and
face enrollment need the models present.

Live log: `build_log.txt` in this folder.
