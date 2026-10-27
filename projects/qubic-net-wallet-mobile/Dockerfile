# syntax=docker/dockerfile:1
# Build the Qubic mobile wallet (.NET MAUI Blazor Hybrid, net10.0-android).
#
# Building an Android MAUI head needs, in addition to the .NET SDK: a JDK, the Android
# SDK (platform + build-tools) with licenses accepted, and the `maui-android` workload.
# Everything is installed here because the audit stage has no network. All managed
# dependencies (Qubic.Core/Services/WalletConnect, MAUI, SQLitePCLRaw/sqlcipher,
# QRCoder, biometric/barcode packages) resolve from nuget.org; the repo's NuGet.config
# pins nuget.org only (the local dev feed is commented out).
FROM mcr.microsoft.com/dotnet/sdk:10.0

ENV DEBIAN_FRONTEND=noninteractive \
    DOTNET_CLI_TELEMETRY_OPTOUT=1 \
    DOTNET_NOLOGO=1 \
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=1 \
    NUGET_XMLDOC_MODE=skip \
    MSBUILDDISABLENODEREUSE=1 \
    JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 \
    ANDROID_HOME=/opt/android-sdk \
    ANDROID_SDK_ROOT=/opt/android-sdk

# JDK 17 + tools needed to fetch and unpack the Android command-line tools.
RUN apt-get update && apt-get install -y --no-install-recommends \
      bash ca-certificates git python3 curl unzip openjdk-17-jdk-headless \
    && rm -rf /var/lib/apt/lists/*

# Android SDK command-line tools + platform-tools. The exact API level and build-tools
# the project needs are installed below by the project's own InstallAndroidDependencies
# target, so we do not hardcode (and chase) a platform version here.
ENV ANDROID_CMDLINE_TOOLS_VERSION=11076708
RUN mkdir -p "$ANDROID_HOME/cmdline-tools" \
    && curl -fsSL -o /tmp/cmdline-tools.zip \
        "https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_CMDLINE_TOOLS_VERSION}_latest.zip" \
    && unzip -q /tmp/cmdline-tools.zip -d "$ANDROID_HOME/cmdline-tools" \
    && mv "$ANDROID_HOME/cmdline-tools/cmdline-tools" "$ANDROID_HOME/cmdline-tools/latest" \
    && rm /tmp/cmdline-tools.zip
ENV PATH="${PATH}:/opt/android-sdk/cmdline-tools/latest/bin:/opt/android-sdk/platform-tools"
RUN yes | sdkmanager --licenses >/dev/null \
    && sdkmanager --install "platform-tools" >/dev/null

# .NET MAUI Android workload.
RUN dotnet workload install maui-android --skip-manifest-update

# Android SDK platform + build-tools the net10.0-android head targets (currently API 36).
# Installed via sdkmanager (not the MSBuild InstallAndroidDependencies target, which crashes
# in a minimal root container). Bump these if the project retargets a newer API level.
RUN yes | sdkmanager --licenses >/dev/null \
    && sdkmanager --install "platforms;android-36" "build-tools;36.0.0" >/dev/null

# scanner contract: the checkout lives inside the image, at /src
COPY . /src
WORKDIR /src

# Restore and build the Android head (Debug keeps asserts and readable IL for analysis).
# All downloads happen here, during the networked build; the audit stage has no network.
RUN dotnet restore Qubic.Net.Wallet.Mobile.sln \
    && dotnet build Qubic.Net.Wallet.Mobile.sln --no-restore -c Debug \
        -p:RunAOTCompilation=false -p:PublishTrimmed=false -m:2 \
    && dotnet build-server shutdown

CMD ["bash"]
