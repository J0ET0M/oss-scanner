# syntax=docker/dockerfile:1
# Build the Qubic desktop wallet (Photino.Blazor).
#
# The app multi-targets net8.0 and net10.0; the .NET 10 SDK compiles both (net8.0
# reference packs restored from NuGet). The Qubic.* dependencies (Qubic.Core,
# Qubic.Services, Qubic.WalletConnect) and Photino.Blazor all resolve from nuget.org,
# which is reachable during this build; the audit stage afterwards has no network.
# The repo's NuGet.config pins nuget.org only (the local dev feed is commented out).
FROM mcr.microsoft.com/dotnet/sdk:10.0

ENV DOTNET_CLI_TELEMETRY_OPTOUT=1 \
    DOTNET_NOLOGO=1 \
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=1 \
    NUGET_XMLDOC_MODE=skip \
    MSBUILDDISABLENODEREUSE=1

RUN apt-get update && apt-get install -y --no-install-recommends \
      bash ca-certificates git python3 \
    && rm -rf /var/lib/apt/lists/*

# scanner contract: the checkout lives inside the image, at /src
COPY . /src
WORKDIR /src

# Restore and build the wallet solution (Debug keeps asserts and readable IL for analysis).
RUN dotnet restore Qubic.Net.Wallet.sln \
    && dotnet build Qubic.Net.Wallet.sln --no-restore -c Debug -m:2 \
    && dotnet build-server shutdown

CMD ["bash"]
