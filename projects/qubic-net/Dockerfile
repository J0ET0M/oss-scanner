# syntax=docker/dockerfile:1
# Build and test the Qubic.Net SDK solution.
#
# Source projects multi-target net8.0 and net10.0; test projects target net10.0.
# The .NET 10 SDK image can compile both target frameworks (net8.0 reference packs
# are restored from NuGet) and runs the net10.0 tests on its bundled runtime.
# All dependencies (incl. the Qubic.Crypto NuGet package) come from nuget.org, which
# is reachable during this build; the audit stage afterwards has no network.
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

# Restore and build the whole solution (Debug keeps asserts and readable IL for analysis).
RUN dotnet restore Qubic.Net.sln \
    && dotnet build Qubic.Net.sln --no-restore -c Debug -m:2 \
    && dotnet build-server shutdown

# Run the test suite so we know the image works; do not abort the build on a test failure.
RUN cd /src && (dotnet test Qubic.Net.sln --no-build -c Debug \
      --results-directory /tmp/test-results --logger "trx" \
      || echo "WARNING: dotnet test reported failures"); dotnet build-server shutdown

CMD ["bash"]
