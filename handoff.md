# Handoff

## Current State

This repository is on the `developer` branch. The latest committed work is:

`9c81cb8 Update Multipaz skills for issuance workflow`

That commit copied the improved `.agents` Multipaz skill from the `issuance` branch into `developer` and validated it. The application implementation work was completed and tested on the `issuance` branch, then the reusable lessons were folded back into `.agents` on `developer`.

## Target

The target was to use and improve the `.agents/skills/multipaz` skill until another agent or developer can reliably reproduce a complete Multipaz holder issuance workflow:

1. Install/configure Multipaz dependencies and verify the setup.
2. Add the minimum holder application dependencies and configuration.
3. Create and securely store a locally generated credential.
4. Add OpenID4VCI issuance so a real credential can be issued to the app.
5. Validate that the issued credential is stored and visible in the holder app.

## What We Built

On the `issuance` branch, the app was updated with:

- Multipaz `0.100.0`.
- `org.multipaz:multipaz`.
- `org.multipaz:multipaz-doctypes`.
- Ktor `3.3.3`.
- Android Ktor engine.
- Darwin Ktor engine.
- Android `INTERNET` permission.
- `openid-credential-offer` deep link support.
- OAuth redirect scheme support.
- Android Multipaz initialization through `initializeApplication(applicationContext)`.
- Local holder wallet based on `DocumentStore` and `SecureArea`.
- Local mock mDL creation.
- OpenID4VCI offer handling.
- Browser OAuth authorization handoff on Android.
- Issued credential storage and display in the app.
- Android host test for local credential creation and storage.

On the `developer` branch, the improved `.agents/skills/multipaz` skill was committed so future agents know the required workflow, failure modes, and validation steps.

## Problems Found

### 1. App accepted the offer but issuing appeared to do nothing

The offer URI filled the app, but tapping `Issue from offer` did not complete issuance.

Root causes found during debugging:

- Multipaz Android context was not initialized before using `Platform` storage.
- Android `INTERNET` permission was missing.
- Ktor client engines were missing for runtime HTTP client creation.
- OAuth authorization-code flow was not wired back into `ProvisioningModel`.
- Public Multipaz issuer rejected arbitrary locally generated wallet attestation keys.

Fixes:

- Added Android initialization with `initializeApplication(applicationContext)`.
- Added `android.permission.INTERNET`.
- Added explicit Android and Darwin Ktor client engines.
- Added custom scheme handling for both credential offers and OAuth redirects.
- Used version-matched sample wallet attestation material for the demo issuer, marked as sample-only.

### 2. Skills did not mention several required issuance details

The original Multipaz skill covered high-level setup but missed important real-world steps.

Fixes added to `.agents/skills/multipaz`:

- Minimum holder architecture.
- Secure local storage pattern.
- Android context initialization.
- Android real-issuer permission and redirect requirements.
- Ktor engine requirement.
- OAuth redirect handoff through `AuthorizationResponse.OAuth`.
- Trusted wallet attestation requirement.
- End-to-end real issuer verification checklist.
- Troubleshooting for common symptoms.

### 3. Automated local credential test exposed timestamp bug

The Android host test initially failed with:

`signedAt cannot have fractional seconds`

Root cause:

- mdoc/MSO timestamps were created directly from `Clock.System.now()`, which can include fractional seconds.

Fix:

- Normalize mdoc/MSO issue and validity timestamps to whole seconds.
- Added this lesson to the skill troubleshooting and secure storage guidance.

### 4. Android host tests cannot use Android Keystore

The host test failed with:

`NoSuchProviderException: no such provider: AndroidKeyStore`

Root cause:

- JVM/Robolectric host tests cannot use the real Android Keystore provider.

Fix:

- Added a test-only path by injecting Multipaz `SoftwareSecureArea`.
- Kept production app behavior using platform `SecureArea`.
- Added this guidance to the skill.

## Branch Notes

- `developer`: contains the committed skill updates in `.agents`.
- `issuance`: contains the app implementation work and validation additions used to prove the workflow.
- `developer` was treated as the working reference during troubleshooting.

## Remaining Limitations

- Demo wallet attestation keys are sample-only and must not be used as production secrets.
- A production wallet should use a registered wallet backend or issuer trust configuration instead of embedded demo attestation material.
