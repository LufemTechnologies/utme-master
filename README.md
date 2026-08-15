# 🎓 UTME Master

> **A professional, LAN-based UTME exam preparation system for Nigerian secondary schools.**  
> Built by [Lufem Technologies](mailto:lufemtechnologies@gmail.com) — Empowering education through technology.

![Version](https://img.shields.io/badge/version-1.0.0-blue) ![Platform](https://img.shields.io/badge/platform-Windows-informational) ![License](https://img.shields.io/badge/license-Commercial-red) ![Status](https://img.shields.io/badge/status-Stable-brightgreen)

---

## 📖 Table of Contents

- [Overview](#overview)
- [Screenshots](#screenshots)
- [What's New in v1.0.0 — Changelog](#whats-new-in-v100--changelog)
- [Tech Stack](#tech-stack)
- [Installation & Setup Guide](#installation--setup-guide)
- [Licensing System](#licensing-system)
- [Known Issues & Roadmap](#known-issues--roadmap)
- [Contributing](#contributing)
- [Contact & Branding](#contact--branding)

---

## Overview

**UTME Master** is a Windows desktop application that allows secondary schools to run UTME (Unified Tertiary Matriculation Examination) practice sessions entirely over a **local area network (LAN)** — no internet required. Students log in from any connected computer in the school, take timed practice exams, and receive instant results. School administrators manage everything from a central control panel.

**Key highlights:**
- Offline-first: works completely without internet after installation
- Multi-subject support (English, Mathematics, Physics, Chemistry, Biology, Economics, and more)
- Hardware-fingerprinted licensing to prevent unauthorized redistribution
- Tiered licensing: Basic, Standard, and Premium tiers for different school sizes
- Secure per-school activation via unique license keys
- Real-time analytics dashboard for teachers and administrators

---

## Screenshots

> 📌 *Screenshots will be added after first public deployment. UI previews available on request.*

Planned screenshot sections:
- **Setup Wizard** — Initial school configuration and licence activation
- **Student Login Screen** — Simple, fast student authentication
- **Exam Interface** — Clean question view with countdown timer
- **Results Dashboard** — Instant score breakdown by subject
- **Admin Control Panel** — Manage students, exams, and school settings

---

## What's New in v1.0.0 — Changelog

This release marks the first stable version of UTME Master. The following critical fixes and new implementations were completed before this release.

---

### 🐛 Critical Bug Fixes

#### 1. Missing `isPrismaClientReady` Function
- **Problem:** The application crashed on startup because `isPrismaClientReady` was called in `main.js` but was never defined, causing an unhandled `TypeError`.
- **Fix:** Implemented `isPrismaClientReady()` as a proper async check that pings the Prisma database connection before any IPC handlers are registered. The app now shows a loading screen with a "Database connecting..." message instead of silently crashing.

#### 2. Broken `resetSetup()` Logic
- **Problem:** Calling `resetSetup()` from the control panel did not properly clear the school configuration. It deleted some config keys but left the `schoolId` and Prisma migration state intact, causing the Setup Wizard to skip steps on re-entry.
- **Fix:** `resetSetup()` now performs a full teardown: clears all `electron-store` keys, drops and re-migrates the Prisma schema, and forces a full restart of the Electron main process after confirmation.

#### 3. Fake Backup Function
- **Problem:** The "Backup Database" button in the admin panel called a stub function that logged `"backup done"` to the console without actually writing any file to disk. Schools using this feature had no real backup.
- **Fix:** Replaced the stub with a real backup routine using Node.js `fs` streams. Backups are saved as timestamped `.sqlite` or `.dump` files in a user-selected directory. A success/failure toast notification now confirms the result.

#### 4. Tiered Licensing Architecture Issues
- **Problem:** The licensing module treated all licence keys identically regardless of tier. A Basic-tier key could unlock Premium features because the tier byte in the key was never validated against the feature access map.
- **Fix:** Rebuilt the licensing flow around `generateTieredKey()` and `verifyTieredKey()`. Each key now encodes the tier, school hardware fingerprint, and expiry date. Feature access is gated at runtime by reading the verified tier from the decoded key — not from the config file.

#### 5. Setup Wizard Resumption Bug
- **Problem:** If a school admin closed the app mid-setup, reopening it would restart the wizard from Step 1 instead of resuming from where they left off. This caused duplicate school records in the database.
- **Fix:** The wizard now writes a `setupProgress` checkpoint to `electron-store` after each completed step. On launch, `main.js` reads this checkpoint and routes directly to the correct step, preventing re-entry to completed steps.

---

### ✨ New Implementations

#### Hardware-Fingerprinted Licensing
Each installation is now bound to the host machine using a composite hardware fingerprint (CPU serial + MAC address + disk serial). License keys are generated per school and will reject activation on any other machine, preventing key sharing between schools.

#### Tiered Licence Key System
Three tiers are now fully implemented:

| Tier | Max Students | Features |
|------|-------------|----------|
| **Basic** | 30 | Core exam taking, basic result view |
| **Standard** | 100 | All Basic + analytics dashboard, subject breakdown |
| **Premium** | Unlimited | All Standard + data export, multi-admin, priority support |

#### Pre-Deployment Installer Pipeline
Built a production installer using **electron-builder** with **Inno Setup**:
- Single `.exe` installer for Windows 7/8/10/11
- Auto-installs Node.js runtime dependency if missing
- Creates Start Menu and Desktop shortcuts
- Registers uninstaller in Windows Add/Remove Programs

#### Prisma Schema Stabilisation
Resolved all Prisma schema-to-database mismatches that were causing silent query failures. Migrations are now run automatically on first launch via `prisma migrate deploy`.

#### Virus Recovery & Dependency Integrity
Recovered from an **Expiro/Win32** malware infection that corrupted `esbuild.exe` inside `node_modules`. Full `node_modules` reinstall was performed and a `.gitignore` rule was enforced to prevent `node_modules` from ever being committed to the repository.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Desktop Shell | Electron |
| Frontend | React + TypeScript |
| Backend / API | Express.js + Node.js |
| Database | PostgreSQL (production) / SQLite (local dev) |
| ORM | Prisma |
| Installer | electron-builder + Inno Setup |
| Licence Crypto | Node.js `crypto` (HMAC-SHA256) |

---

## Installation & Setup Guide

### Requirements

- Windows 7 / 8 / 10 / 11 (32-bit or 64-bit)
- Minimum 4 GB RAM, 2 GHz processor
- 500 MB free disk space
- LAN network (wired recommended for exam sessions)
- A valid UTME Master licence key (contact Lufem Technologies)

### Step 1 — Download the Installer

Download the latest release from the [Releases page](../../releases) of this repository.

```
UTMEMaster-Setup-v1.0.0.exe
```

### Step 2 — Run the Installer

1. Double-click `UTMEMaster-Setup-v1.0.0.exe`
2. Accept the licence agreement
3. Choose an installation directory (default: `C:\Program Files\UTME Master`)
4. Click **Install** and wait for completion
5. Click **Finish** — the Setup Wizard will launch automatically

### Step 3 — Setup Wizard

The Setup Wizard will walk you through:

1. **School Information** — Enter school name, address, and administrator credentials
2. **Database Initialisation** — Automatic; no action required
3. **Licence Activation** — Enter the licence key provided by Lufem Technologies
4. **Network Configuration** — Set the server port (default: `3000`) and confirm LAN IP
5. **Done** — The Control Panel opens

### Step 4 — Student Access

Students access UTME Master from any computer on the same network by opening a browser and navigating to:

```
http://<server-ip>:3000
```

The server IP is displayed on the admin Control Panel home screen.

---

## Licensing System

UTME Master uses a **hardware-fingerprinted, tiered licensing** system to ensure each installation is unique to the purchasing school.

### How It Works

1. When you activate UTME Master, the app generates a **hardware fingerprint** from your server machine (CPU + MAC address + disk serial).
2. This fingerprint is submitted to Lufem Technologies (offline-capable via manual activation).
3. A **licence key** is issued that is cryptographically tied to your machine and your chosen tier.
4. The key is verified locally on every app launch — no internet check after activation.

### Licence Key Format

```
LT-[TIER]-[HARDWARE_HASH]-[EXPIRY]-[CHECKSUM]
```

Keys are validated using HMAC-SHA256 with a Lufem Technologies private salt. Tampered or copied keys will fail verification.

### Renewing or Upgrading

Contact Lufem Technologies with your school name and current hardware fingerprint (visible in **Settings > Licence Info**) to receive a renewal or upgrade key.

---

## Known Issues & Roadmap

### Known Issues (v1.0.0)

| Issue | Severity | Status |
|-------|----------|--------|
| Backup restore from `.dump` not yet implemented | Medium | Planned for v1.1 |
| Admin password reset requires manual DB edit | Low | Planned for v1.1 |
| Offline question bank import limited to CSV only | Low | XLSX support planned |
| No automatic LAN IP detection on multi-NIC servers | Low | Workaround: manually set IP in config |

### Roadmap

#### v1.1 — Stability & School UX
- [ ] Backup restore UI
- [ ] Admin self-service password reset
- [ ] XLSX question import support
- [ ] Auto-detect LAN IP on multi-NIC machines
- [ ] Student result printing (PDF export)

#### v1.2 — Analytics Expansion
- [ ] Per-student performance history
- [ ] Class-level score comparison charts
- [ ] Teacher subject-drill-down reports
- [ ] Exportable CSV results for WAEC/NECO record-keeping

#### v2.0 — Cloud Sync (Optional Add-on)
- [ ] Optional cloud backup to Lufem Technologies server
- [ ] Remote licence management portal for school admins
- [ ] Multi-branch school support

---

## Contributing

UTME Master is a **commercial product** developed by Lufem Technologies. The source code in this repository is maintained internally.

External contributions are not accepted at this time. However, if you are:
- A school IT administrator reporting a bug
- A developer interested in a partnership
- A distributor interested in reselling UTME Master

Please reach out via the contact details below.

## Contact & Branding

<div>

**Lufem Technologies**  
Department of Computer Science & IT  
Lufem College of Technology, Agege, Lagos, Nigeria

📧 Email: [ayokule234@gmail.com](mailto: ayokule234@gmail.com)  
🌐 Website: [https://utme-master.netlify.app/] 
📦 GitHub: [github.com/LufemTech](https://github.com/Ayokule)

</div>

---

> *UTME Master is an independent product of Lufem Technologies and is not affiliated with, endorsed by, or connected to JAMB (Joint Admissions and Matriculation Board) in any official capacity.*

---

<div align="center">
  <sub>Built with ❤️ for Nigerian secondary schools · © 2026 Lufem Technologies · All rights reserved</sub>
</div>
