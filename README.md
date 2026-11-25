
name: Build JANSEVA APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Extract JANSEVA project
        run: |
          mkdir source
          ZIP_FILE=$(find . -maxdepth 1 -name '*.zip' -print -quit)
          unzip -q "$ZIP_FILE" -d source
          PROJECT_DIR=$(dirname "$(find source -name settings.gradle.kts -print -quit)")
          echo "PROJECT_DIR=$PROJECT_DIR" >> $GITHUB_ENV

      - name: Configure Android SDK
        run: |
          echo "ANDROID_HOME=$ANDROID_HOME"
          SDKMANAGER=$(find "$ANDROID_HOME/cmdline-tools" -name sdkmanager -type f | sort -V | tail -n 1)
          chmod +x "$SDKMANAGER"

          yes | "$SDKMANAGER" --licenses || true

          "$SDKMANAGER" "platforms;android-35" "build-tools;35.0.0"

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'

      - name: Build JANSEVA APK
        run: |
          gradle --no-daemon -p "$PROJECT_DIR" assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: JANSEVA-APK
          path: source/**/app/build/outputs/apk/debug/*.apk
