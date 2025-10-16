# Walt — Architecture & Design

**Walt** is a privacy-first digital wallet for Android that allows users to securely load and use their payment cards for tap-to-pay — without relying on Google Wallet or Apple Pay.

Walt’s mission is to make **private, independent mobile payments** possible.
Users can sign up at [**walt.is**](https://walt.is) to join the waitlist and learn about progress toward the first public release.

---

## Intent

Most mobile payments today depend on closed ecosystems — like Google Wallet or Apple Pay — that mediate between the user, bank, and network. Walt takes a different approach:

- **Direct-to-aggregator model:** Walt integrates directly with PCI compliant payment aggregators to handle tokenization and DPAN issuance from Visa and Mastercard.
- **On-device data:** All API calls are made directly from the device to the aggregator, keeping the architecture lean and privacy-preserving. Walt does not host any centralized processing or databases.
- **User-controlled data:** Card data never passes through Walt’s own servers; encryption and eligibility checks happen between the app and Mastercard/Visa via the aggregator.
- **HCE-based payments:** Walt uses Android’s **Host Card Emulation (HCE)** APIs to enable NFC tap-to-pay functionality with virtual cards.

This architecture gives users modern mobile payment capability without locking them into a single vendor ecosystem.

---

## How Walt Works

1. **Card Loading**
   Users manually enter their PAN in the Walt app.
   The app calls the aggregator’s `/eligibility` API to validate the card and request a DPAN.
   Once approved, the DPAN is stored securely on the device for use in NFC transactions.
   Find more details in [Card Loading Process](docs/card-loading.md).

2. **Tokenized Payments**
   During tap-to-pay, the terminal sends metadata (amount, merchant ID, nonce) to the phone.
   Walt signs this data using cryptographic keys associated with the DPAN and forwards the payload to the aggregator for authorization.
   Find more details in [NFC Transactions](docs/nfc-transactions.md).

3. **Secure Storage**
   Keys and tokens are held in Android’s secure vault (TEE-backed Keystore), isolated from the app layer.
   Find more details in [Secure Storage](docs/secure-storage.md).

4. **Privacy by Design**
   Walt does not collect transaction data, behavioral data, or analytics beyond what’s needed for security and compliance.
   Find more details in [Privacy by Design](docs/privacy-by-design.md).

---

## Status

Walt is in the early architectural design phase.
This repository will evolve as the app, integrations, and partnerships are developed.

Join the waitlist at **[walt.is](https://walt.is)** for updates.

---

© 2025 Walt. All rights reserved.
