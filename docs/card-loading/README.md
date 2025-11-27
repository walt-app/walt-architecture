# Card Loading Process

The card loading process is how users add their physical payment cards to Walt for tap-to-pay use. This involves several security steps to ensure the user is authorized to provision the card, and that the card is eligible for tokenization.

**Last Updated**: 2025-11-27

## Overview

Before any card loading can occur, the MTP SDK must be initialized on app launch. Then, when a user manually enters their card details (PAN, expiry date, and CVC), Walt coordinates with a payment aggregator to:

0. **Initialize MTP SDK (every app launch)** — SDK performs device security checks (root detection, security posture, battery state). If this fails, Walt blocks ALL functionality.
1. **Validate eligibility** — Check if the card is supported and the device meets security requirements
2. **Perform step-up verification (if needed)** — Request additional authentication via OTP or out-of-band methods
3. **Provision a device-bound token (DPAN)** — Create a tokenized version of the card that's securely tied to the specific device

Throughout this process, sensitive card data is encrypted and never passes through Walt's servers. All communication happens directly between the Walt app on the user's device and the payment aggregator's API.

## Responsibility Split: SDK vs Walt

| Responsibility | Owner |
|----------------|-------|
| Device security checks (root, battery, security posture) | **MTP SDK** (via `/initialize`) |
| Play Integrity attestation | **Walt** |
| PAN encryption | **MTP SDK** |
| Eligibility, verification, provisioning network calls | **MTP SDK** |
| UI for card entry, verification, success/error | **Walt** |

## Sequence Diagram

The following diagram shows the optimal manual card load flow, including local preflight checks, eligibility validation, optional verification, and token provisioning:

```mermaid
sequenceDiagram
  autonumber
  participant App as Walt App (Android)
  participant SDK as MTP SDK
  participant Agg as Aggregator / TSP API
  participant TSP as Card Network (VTS / MDES)

  Note over App,SDK: Step 0: SDK Initialization (every app launch)
  App->>SDK: initialize()
  SDK->>SDK: Root detection, security checks, battery state
  alt initialization fails
    SDK-->>App: Failure (detailed error)
    App->>App: BLOCK ALL FUNCTIONALITY, show error
  else initialization succeeds
    SDK-->>App: Success
  end

  Note over App: Step 1: Local preflight (during provisioning)
  App->>App: Check NFC/HCE & default wallet
  App->>App: Play Integrity attestation (Walt's responsibility)
  App->>App: Collect PAN/expiry/CVC (manual)
  App->>SDK: Encrypt PAN

  %% Eligibility
  App->>Agg: POST /eligibility (encryptedPAN, attestation, device info)
  Agg->>Agg: Validate eligibility, issuer policy, risk checks
  Agg-->>App: {cardRef, status: ELIGIBLE | PENDING_VERIFICATION, methods:[OTP,OOB]}

  %% Step-up verification (if needed)
  alt status == PENDING_VERIFICATION
    App->>Agg: POST /verification/start {cardRef, method}
    Agg->>Agg: Create OTP or OOB challenge with issuer
    Agg-->>App: {verificationSessionId}

    App->>App: User enters OTP or completes OOB
    App->>Agg: POST /verification/confirm {sessionId, code?}
    Agg->>Agg: Validate challenge
    Agg-->>App: {verified:true}
  end

  %% Provisioning
  App->>App: Generate Keystore keypair & attestation
  App->>Agg: POST /provision {cardRef, keyAttestation, walletId/TRID}
  Agg->>TSP: Create device-bound token (DPAN)
  TSP-->>Agg: {tokenRef, maskedDPAN, tokenState}
  Agg-->>App: {tokenRef, maskedDPAN, tokenState}

  App->>App: Store token securely (Keystore / SDK)
  App->>App: UI → "Ready for tap-to-pay"
```

## Key Steps Explained

### 0. SDK Initialization (Every App Launch)

Before any card loading can occur, Walt must initialize the MTP SDK on every app launch:

```kotlin
when (val result = mtpSdk.initialize(context)) {
    is Success -> {
        // Proceed to normal app flow
    }
    is Failure -> {
        // BLOCK ALL APP FUNCTIONALITY
        // Display SDK's detailed error message to user
        showBlockingError(result.error.message)
    }
}
```

**What the SDK checks**:
- Root/jailbreak detection
- Device security posture
- Battery state (anti-fraud measure)
- Other device-level security checks

**What the SDK does NOT check**:
- Play Integrity attestation (Walt's responsibility - see Step 1)

**On failure**: Walt must block ALL functionality and display the SDK error. No card provisioning or payments are allowed on devices that fail security checks.

### 1. Local Preflight Checks (During Provisioning)

Before making any network calls for card provisioning, Walt performs local validation:
- **NFC/HCE availability**: Ensures the device supports Host Card Emulation
- **Play Integrity attestation**: Walt requests attestation token (SDK does NOT handle this)
- **Data encryption**: Encrypts the PAN using the MTP SDK

### 2. Eligibility Check

The app sends the encrypted PAN along with device attestation data to the aggregator's `/eligibility` endpoint. The aggregator:
- Validates the card is eligible for tokenization
- Checks issuer policies and risk parameters
- Returns a card reference and eligibility status

### 3. Step-Up Verification (Conditional)

If the issuer requires additional verification, the aggregator returns a `PENDING_VERIFICATION` status. Walt then:
- Calls `/verification/start` to initiate an OTP or out-of-band challenge
- Prompts the user to complete verification
- Confirms verification via `/verification/confirm`

### 4. Token Provisioning

Once eligibility is confirmed, Walt:
- Generates a cryptographic keypair in Android Keystore (hardware-backed when available)
- Sends the key attestation to the aggregator
- The aggregator coordinates with the card network (Visa Token Service or Mastercard MDES) to create a DPAN
- The DPAN is returned to the app and stored securely in Keystore

### 5. Ready for Payments

The token is now ready for NFC tap-to-pay transactions. The user sees confirmation in the UI, and the DPAN remains securely isolated in the device's trusted execution environment (TEE).

## Security Considerations

- **SDK device checks**: MTP SDK validates device security (root, battery, security posture) on every app launch
- **Play Integrity**: Walt separately handles Play Integrity attestation for provisioning
- **End-to-end encryption**: PAN data is encrypted on-device before transmission
- **No server-side storage**: Walt never sees or stores unencrypted card data
- **Device binding**: The DPAN is cryptographically tied to the device's hardware-backed keystore
- **Attestation**: Play Integrity and key attestation prove device authenticity to the aggregator
- **Fail-fast on insecure devices**: SDK initialization failure blocks ALL Walt functionality
