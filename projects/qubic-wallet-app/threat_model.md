# Threat model — Qubic Wallet App (mobile)

## What this project does and where untrusted input enters
Qubic Wallet App is the official mobile wallet: a Flutter/Dart app (Android + iOS, with
desktop/web targets present) that generates and stores seeds, derives identities, signs
and broadcasts transactions, and connects to dApps via WalletConnect. It guards
**private keys and user funds**, so confidentiality of secrets and integrity of what the
user signs ("what you see is what you sign") are the whole game.

Untrusted input enters through:
- **Network/RPC and HTTP API responses** rendered to the user (balances, tick status,
  transaction history, assets) — a malicious or compromised node/RPC/backend can return
  crafted data.
- **WalletConnect requests** from remote dApps (pairing/connection proposals,
  sign/transaction requests) — attacker-controlled (see `walletconnect.md`).
- **QR codes, deep links / app links, pasted addresses and amounts, and imported
  seed/vault material** — user-supplied but may be adversarial.
- **Device-local secret storage** (secure storage / keystore / keychain) and any local
  files or shared-preferences the app reads at startup.

## Components that matter most / least
- **Most critical:** seed/private-key storage and handling (at rest in secure storage and
  in memory), key derivation, and the signing path — what exactly gets signed vs. what
  the UI shows the user.
- **High:** WalletConnect request handling and the user-consent flow; address/amount
  parsing and display (recipient/amount substitution or confusable/Unicode rendering);
  deserialization of node/RPC/backend responses; QR/deep-link handling; the Dart↔native
  plugin boundary (secure storage, cryptography, biometrics) and Android `intent-filter` /
  iOS URL-scheme entry points.
- **Lower priority:** cosmetic UI, l10n strings, settings that do not touch keys or
  signing.

## How to exercise it
- The image pins Flutter 3.38.3 (matching `.fvmrc`), runs `flutter pub get`, and builds
  the Android APK (`flutter build apk --debug`) — the real shipped artifact — which
  compiles all Dart and the Android plugin layer. The wallet logic lives under `lib/`;
  cryptography and secure storage are delegated to the Flutter crypto/secure-storage
  plugins it depends on (`pubspec.yaml` / `pubspec.lock`).
- Treat every network response and every WalletConnect/deep-link/QR input as
  attacker-controlled, and focus on the signing/consent path and on anywhere a remote- or
  user-derived value reaches the signer or is persisted alongside secrets.
