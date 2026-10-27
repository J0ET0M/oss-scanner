# Threat model — Qubic.Net

## What this project does and where untrusted input enters
Qubic.Net is the official .NET SDK that applications (wallets, explorers, services, bots)
use to talk to the Qubic network. It is distributed as the `Qubic.*` NuGet packages, so a
defect here propagates to every downstream consumer. The library both **produces** data
(signing, serialization) and **consumes untrusted data** from the network.

Untrusted input enters through:
- **Wire deserialization** of peer/node responses and packets (`Qubic.Serialization`,
  `Qubic.Network`): message headers, tick/transaction/asset structures, variable-length fields.
- **RPC / HTTP responses** from Qubic RPC endpoints and `Qubic.Services` (JSON and binary
  payloads from servers that may be malicious or compromised).
- **WalletConnect** session/request payloads (`Qubic.WalletConnect`) originating from peers.
- **User/application-supplied** seeds, identities, amounts and contract inputs passed into the API.

## Components that matter most / least
- **Most critical:** `Qubic.Crypto` (KangarooTwelve, FourQ signing/verification, seed→key
  derivation, identity encoding) — correctness and constant-time behavior; `Qubic.Serialization`
  and `Qubic.Network` length/bounds handling when decoding attacker-controlled bytes.
- **High:** `Qubic.Core` transaction construction and amount/field handling; `Qubic.Rpc`
  and `Qubic.Services` response parsing; `Qubic.WalletConnect` request handling.
- **Lower priority:** `Qubic.Bob`, `Qubic.ChainAnalytics` and the `tools/` CLIs.

## How to exercise it
- `tests/**` contains the xUnit suites per package; the image builds the solution and runs
  `dotnet test`. `src/Qubic.Serialization` and `src/Qubic.Network` are the right entry points
  for malformed-input reasoning; `src/Qubic.Crypto` for cryptographic correctness.

## How you rate severity
- A cryptographic flaw — signature forgery/bypass, faulty verification, seed/private-key
  leakage, predictable key derivation, or a non-constant-time secret comparison — is **critical**.
- Memory/`unsafe` corruption or an unchecked length/allocation reachable from deserializing
  network or RPC data is **high** (crash/DoS) and **critical** if it enables code execution or
  key disclosure in a consuming app.
- A parsing defect that lets a malicious server/peer make the SDK return wrong-but-plausible
  data (e.g. a mismatched balance, recipient or amount surfaced to a wallet) is **high**.
- Unhandled-exception DoS from malformed input is **medium** unless it is trivially remote.

## Anything to leave alone
- Issues that require the application itself to pass obviously invalid local arguments (misuse
  of the API by the integrator, not attacker-reachable) are low/out of scope.
