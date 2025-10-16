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
   The app calls the aggregator’s `/eligibility` API to validate the card and request a DPAN.
   Once approved, the DPAN is stored securely on the device for use in NFC transactions.

   Find more details in [Card Loading Process](docs/card-loading.md).

2. **Tap-To-Pay Transactions**
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
