# Threat model — Qubic Wallet (web / Electron)

## What this project does and where untrusted input enters
Qubic Wallet is the official web wallet: an Angular 16 PWA (also packaged as an Electron
desktop app via `forge.config.js`) that generates and stores seeds in the browser,
derives identities, signs transactions and broadcasts them over RPC. It guards
**private keys and user funds**, so confidentiality of secrets and integrity of what the
user signs ("what you see is what you sign") are the whole game.

Untrusted input enters through:
- **Network/RPC and HTTP API responses** rendered to the user (balances, tick status,
  transaction history, asset lists) — a malicious or compromised node/RPC/backend can
  return crafted data.
- **Pasted/scanned addresses and amounts, and imported seed/vault files** — user-supplied
  but may be adversarial (a malicious recipient address, a crafted import/vault file).
- **In-browser storage** (IndexedDB/localStorage) and, for the Electron build, any
  local on-disk config/vault files read at startup.
- **The rendering/DOM surface** itself: any RPC- or user-derived string shown in the UI
  is a potential injection vector (template binding, `innerHTML`, deep links).

## Components that matter most / least
- **Most critical:** seed/vault storage and handling (at rest in the browser and in
  memory), key derivation, and the signing path — what exactly gets signed vs. what the
  UI shows the user.
- **High:** address/amount parsing and display (recipient/amount substitution or
  confusable/Unicode rendering); deserialization and rendering of node/RPC/backend
  responses; vault import/export; for Electron, the main/renderer boundary
  (nodeIntegration, contextIsolation, IPC) and any custom protocol/deep-link handlers.
- **Lower priority:** cosmetic UI, i18n/transloco strings, settings that do not touch
  keys or signing.

## How to exercise it
- The image installs dependencies with Yarn 1 (from `yarn.lock`) and runs the Angular
  production build (`yarn build`). The wallet logic (seed handling, signing, RPC) lives
  under `src/`; cryptography is delegated to the Qubic crypto libraries it depends on.
- There is no server component in this repo — it is a client that trusts no remote
  endpoint with secrets, so the review should assume every network response is
  attacker-controlled and focus on the signing/consent path and on any place an
  RPC- or user-derived value reaches the DOM or the signer.
