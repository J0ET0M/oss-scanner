# Threat model — Qubic.Net.Wallet (desktop)

## What this project does and where untrusted input enters
Qubic.Net.Wallet is the desktop wallet: a Photino.Blazor app that generates and stores
seeds, derives identities, signs and broadcasts transactions, and connects to dApps via
WalletConnect. It guards **private keys and user funds**, so confidentiality of secrets
and integrity of what the user signs are the whole game.

Untrusted input enters through:
- **Network/RPC responses** rendered to the user (balances, tick status, transaction
  history) — a malicious or compromised node/RPC can return crafted data.
- **WalletConnect requests** from remote dApps (connection proposals, sign/transaction
  requests) — attacker-controlled.
- **QR codes, pasted addresses/amounts, and imported seed/key material** — user-supplied
  but may be adversarial (e.g. a malicious address or a crafted import file).
- **Any local on-disk wallet/config files** the app reads at startup.

## Components that matter most / least
- **Most critical:** seed/private-key storage and handling (at rest and in memory),
  key derivation, and the signing path — what exactly gets signed vs. what the UI shows
  the user ("what you see is what you sign").
- **High:** WalletConnect request handling and user-consent flow; address/amount parsing
  and display (recipient/amount substitution or confusable rendering); deserialization of
  node/RPC responses.
- **Lower priority:** cosmetic UI, settings that do not touch keys or signing.

## How to exercise it
- The image restores and builds `Qubic.Net.Wallet.sln` with the .NET 10 SDK. The wallet
  logic (seed handling, signing, WalletConnect) is in the Blazor component/service code;
  cryptography is delegated to the `Qubic.*` SDK packages (reviewed separately as qubic-net).

## How you rate severity
- Any path that **exfiltrates or logs a seed/private key**, writes it in cleartext where it
  should not be, or leaves it recoverable is **critical**.
- Any path where the **transaction actually signed differs from what the user approved**
  (recipient, amount, contract call) — including confusable/overflowed UI rendering or a
  WalletConnect request that signs without clear consent — is **critical**.
- A malicious dApp/RPC causing fund loss without explicit, accurate user consent is **critical**;
  causing a crash/DoS only is **medium**.
- Weak randomness in seed generation, or predictable/reused nonces in signing, is **critical**.

## Anything to leave alone
- Threats requiring malware already running as the user (full local compromise, memory
  scraping by another process) are generally out of scope — assume the host OS is not already
  owned. Report secret-at-rest weaknesses that make such compromise materially worse, though.
- Cryptographic primitive bugs belong to the qubic-net SDK; here, focus on how the wallet
  *uses* them and how it handles secrets and consent.
