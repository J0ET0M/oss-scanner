# Threat model — Qubic.Net.Wallet.Mobile (Android)

## What this project does and where untrusted input enters
The Android Qubic wallet (.NET MAUI Blazor Hybrid). It generates and stores seeds,
derives identities, signs and broadcasts transactions, unlocks with biometrics, stores
data in an encrypted SQLCipher database, and scans QR codes / handles WalletConnect.
It guards **private keys and user funds** on a device that may be shared, lost, or stolen.

Untrusted input enters through:
- **QR codes** scanned by the camera (addresses, amounts, WalletConnect URIs, payment requests).
- **WalletConnect requests** from remote dApps (connection, sign, transaction requests).
- **Network/RPC responses** from Qubic nodes rendered to the user.
- **Imported seed/key material** and pasted addresses/amounts.
- **Android IPC**: deep links / intents, and any exported component the app registers.
- **The local SQLCipher database and preferences** restored across launches.

## Components that matter most / least
- **Most critical:** seed/private-key generation, storage at rest (SQLCipher key management,
  keystore/biometric gating) and in-memory handling; the signing path and what is actually
  signed vs. shown to the user.
- **High:** biometric/unlock gate (can it be bypassed? is the DB key tied to it correctly?);
  WalletConnect consent flow; QR/address/amount parsing and display (recipient/amount
  substitution, confusable rendering); RPC response deserialization.
- **Lower priority:** cosmetic UI, non-security settings.

## How to exercise it
- The image installs the Android SDK + MAUI workload and builds
  `Qubic.Net.Wallet.Mobile.sln` for `net10.0-android`. Wallet logic (seed handling, signing,
  storage, WalletConnect) lives in the shared C#/Blazor code; cryptography is delegated to the
  `Qubic.*` SDK packages (reviewed separately as qubic-net).

## How you rate severity
- Any path that **exfiltrates, logs, or stores a seed/private key in recoverable cleartext**,
  or that weakens the SQLCipher key so DB contents are recoverable off-device, is **critical**.
- Any path where the **signed transaction differs from what the user approved** (recipient,
  amount, contract call) — via QR/WalletConnect/UI — is **critical**.
- A **biometric/lock-screen bypass** that exposes keys or allows signing is **critical**.
- Weak randomness in seed generation or predictable/reused signing nonces is **critical**.
- An exported Android component, deep link, or intent that lets another app trigger signing,
  read wallet data, or drive the wallet without consent is **high/critical** by impact.
- A malformed QR/RPC/WalletConnect payload that only crashes the app is **medium**.

## Anything to leave alone
- Threats requiring a rooted device or malware already running with elevated privileges are
  generally out of scope — but do report at-rest weaknesses that make such compromise worse
  (e.g. keys not actually protected by the keystore/biometric).
- Cryptographic primitive bugs belong to the qubic-net SDK; here, focus on how the wallet
  *uses* them and how it protects secrets, storage, and user consent on-device.
