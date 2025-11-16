# Walt — Architecture & Design

**Walt** is a privacy-first digital wallet for Android that allows users to securely load and use their payment cards for tap-to-pay without relying on Google Wallet.

Walt’s mission is to make **private, independent mobile payments** possible.
Users can sign up at [**walt.is**](https://walt.is) to join the waitlist and learn about progress toward the first public release.

---

## Intent

Walt exists to give people an alternative to Google Wallet — a private, open, and independent way to make tap-to-pay payments on Android.

We believe your spending data is sacred.
Walt never stores your card data, transactions, or behavior on our servers.
Everything that matters lives encrypted on your own phone.

There’s no AI, no data mining, no ads — just a simple flow:

---

## How Walt Works

1. **Card Loading**
   Users manually enter their PAN in the Walt app.
   Walt uses **MeaWallet's Mobile Token Platform (MTP)** to handle card digitization and tokenization.
   The MTP SDK validates eligibility, coordinates with card networks (Visa/Mastercard), and provisions a secure DPAN on the device.
   Once approved, the DPAN is stored securely on the device for use in NFC transactions.

   Find more details in [Card Loading Process](docs/card-loading/README.md).

2. **Tap-To-Pay Transactions**
   During tap-to-pay, the terminal communicates via NFC using EMV Contactless protocols.
   Walt's MTP SDK handles Host Card Emulation (HCE), generating cryptograms using keys associated with the DPAN.
   The transaction is authorized through the card network without Walt servers being involved.

   Find more details in [NFC Transactions](docs/nfc-transactions/README.md).

3. **Secure Storage**
   Keys and tokens are held in Android’s secure vault (TEE-backed Keystore), isolated from the app layer.

   Find more details in [Secure Storage](docs/secure-storage/README.md).

4. **Privacy by Design**
   Walt does not collect transaction data, behavioral data, or analytics beyond what’s needed for security and compliance.

   Find more details in [Privacy by Design](docs/privacy-by-design/README.md).

---

## Status

Walt is in **active development**. The Android app is being built with a multi-module Clean Architecture approach using Kotlin, Jetpack Compose, and the MeaWallet MTP SDK.

**Current Phase**: Phase 1 - Foundation & Core Infrastructure (19/28 stories complete)

### Implementation Details

- **[Project Structure](docs/project-structure/)** - Multi-module organization and dependencies
- **[Technology Stack](docs/technology-stack/)** - Technology choices and rationale

Join the waitlist at **[walt.is](https://walt.is)** for updates.

---

© 2025 Walt. All rights reserved.
