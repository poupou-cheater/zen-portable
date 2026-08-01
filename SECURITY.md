# Security & Cryptographic Architecture

This document details the security model, anti-malware (anti-stealer) mitigations, key derivation hierarchy, and technical architecture for this custom Portable Zen Browser build.

---

## 1. Architecture & Execution Overview

```
[USB Drive] ──> Seed / Passphrase Entry (Works on Any Host PC)
                      │
                      ▼
            [Argon2id + AES-256]
                      │
                      ▼
        [Decrypted DB in Memory]
                      │
                      ├──> Apply Selected Options (Obfuscation, PC Check, Workspaces)
                      └──> Launch Browser (Obfuscated Profile / Exe)
```

---

## 2. Launch-Only Key Derivation & Encryption

Encryption and key derivation are performed strictly during startup to eliminate runtime CPU and RAM overhead.

* **Launch-Time Decryption:** Master key derivation (**Argon2id** or **PBKDF2-HMAC-SHA256**) and custom **AES-256** decryption run exclusively at browser launch.
* **No Disk Keyfiles:** No raw decryption keys or keyfiles are ever stored on disk.
* **Zero Runtime Overhead:** Once initialized, there is zero CPU or memory overhead for cryptographic operations during browsing.
* **Reinforced Offline Protection:** High-cost key derivation parameters (memory-hard Argon2id settings) mathematically block offline brute-force attacks against the encrypted database.

---

## 3. Modular Unlock Modes (User-Defined Choice)

Users can select their preferred unlock mechanism depending on their threat model:

### Option A: Pure Master Key / Seed (Recommended Default)
* **How it works:** The database is decrypted strictly via passphrase/seed at launch. No hardware or host identifiers are used.
* **Pros:** 100% portable on any host PC, zero trace left on the host system, zero complexity.
* **Cons:** Requires entering the passphrase every time the browser is opened.

### Option B: Obfuscated Memory Key (Anti-RAM Scraping)
* **How it works:** Key is derived on launch, used to open the DB, then either immediately wiped from RAM or obfuscated using an ephemeral XOR mask while running.
* **Pros:** Protects against host-level memory-dumping malware/stealers running in the background.
* **Cons:** Slightly higher RAM management logic required in the wrapper code.

### Option C: Remember Key & Obfuscated Token (Configurable Expiration & PC Hash Binding)
* **How it works:** On initial launch, the user unlocks via Master Key / Seed. The session key can then be cached in an obfuscated format so subsequent launches auto-unlock without requiring passphrase entry every time.
* **Configurable Auto-Deletion (TTL):** Users can set an automatic expiration policy for the stored key (e.g., auto-delete after 1 hour, 24 hours, 7 days, or never).
* **Optional PC Hash Binding:** Users can choose to link the cached key strictly to the host machine's hardware hash. If the USB drive is plugged into a different PC, the cached key is invalidated and purged, enforcing seed/passphrase entry.
* **Pros:** Maximum convenience on personal/trusted devices while retaining strict expiration and hardware-binding safety nets.
* **Cons:** Stores an obfuscated token on disk (protected by expiration and hardware binding), slightly increasing physical exposure compared to pure memory-only entry.

---

## 4. Pragmatic Anti-Stealer & Evasion Mitigations

Infostealers (*RedLine, Vidar, Lumma, etc.*) target standard paths, default process names, and Windows DPAPI credential stores.

* **Custom Binary & Directory Obfuscation:** Executable names and user profile directories are customized and decoupled from default Firefox/Zen naming conventions (avoiding `%APPDATA%\ZenBrowser`).
* **Bypassing Infostealer Scrapers:** Breaking default naming conventions neutralizes automated file-scraping malware heuristics targeting browser credentials.
* **DPAPI Decoupling:** Browsing data is completely independent of Windows DPAPI, ensuring true portability and security across untrusted host systems.

---

## 5. Transactional Storage & Flash Wear Protection

* **Transactional SQLite Engine (WAL / Snapshots):** The storage backend utilizes transactional SQLite with **Write-Ahead Logging (WAL)** and atomic snapshotting.
* **USB Flash Lifespan:** Minimizes write cycles to prevent premature wear on USB flash storage.
* **Dirty Unmount Resilience:** Guarantees database integrity and prevents file corruption in the event of a sudden USB drive disconnection.
