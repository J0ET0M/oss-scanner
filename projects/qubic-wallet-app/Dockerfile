# syntax=docker/dockerfile:1
# Build the official Qubic mobile wallet (Flutter/Dart).
#
# The repo pins Flutter 3.38.3 (.fvmrc; Dart SDK >=3.0.0). We build the Android APK —
# the real shipped artifact — which exercises the Dart compile plus the Android plugin
# layer. The Gradle build is AGP 8.9.0 / Gradle 8.13 (needs JDK 17); minSdk 21, while
# compileSdk/targetSdk/ndkVersion come from Flutter's defaults for this version.
#
# All of pub.dev, the Flutter SDK, the Android command-line tools and any SDK
# platform/build-tools/NDK that the Gradle build pulls resolve over the network, which is
# available during this build; the audit stage afterwards has no network. We accept the
# Android SDK licenses and let AGP provision the exact platform/build-tools/NDK it needs.
# A --debug APK is built so no signing keystore is required.
FROM debian:bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
      bash ca-certificates curl git unzip zip xz-utils \
      openjdk-17-jdk-headless libglu1-mesa \
    && rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# --- Flutter 3.38.3 (pinned to match .fvmrc) ---
ENV FLUTTER_HOME=/opt/flutter
RUN git clone --depth 1 --branch 3.38.3 https://github.com/flutter/flutter.git "$FLUTTER_HOME"
ENV PATH="$FLUTTER_HOME/bin:$FLUTTER_HOME/bin/cache/dart-sdk/bin:${PATH}"
RUN git config --global --add safe.directory "$FLUTTER_HOME" \
    && flutter config --no-analytics --no-cli-animations >/dev/null 2>&1 || true \
    && flutter --version

# --- Android SDK (command-line tools + a baseline platform/build-tools) ---
ENV ANDROID_HOME=/opt/android-sdk
ENV ANDROID_SDK_ROOT=/opt/android-sdk
ENV PATH="$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:${PATH}"
RUN mkdir -p "$ANDROID_HOME/cmdline-tools" && cd /tmp \
    && curl -fsSL -o clt.zip https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip \
    && unzip -q clt.zip -d "$ANDROID_HOME/cmdline-tools" \
    && mv "$ANDROID_HOME/cmdline-tools/cmdline-tools" "$ANDROID_HOME/cmdline-tools/latest" \
    && rm clt.zip
RUN yes | sdkmanager --licenses >/dev/null \
    && sdkmanager --install "platform-tools" "platforms;android-36" "build-tools;36.0.0" >/dev/null
RUN flutter precache --android

# scanner contract: the checkout lives inside the image, at /src
COPY . /src
WORKDIR /src

# Resolve Dart/Flutter packages, then build the Android APK. AGP auto-provisions any
# extra SDK platform/build-tools/NDK it needs (licenses already accepted above).
RUN flutter pub get \
    && flutter build apk --debug

CMD ["bash"]
