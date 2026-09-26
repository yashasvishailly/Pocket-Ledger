# Pocket Ledger, planned architecture

## Status

A private Android prototype is in progress at v0.3.6: an on-device SMS ledger, not a Play Store release or a public APK. This document records the intended boundaries and data flow. It is not a component-by-component claim about that build.

v0.3.6 imports SMS, manages accounts, re-checks imported messages, blocks screenshots, and can export a CSV by year. Records stay on the device. Uninstall deletes the ledger. Backup is not implemented.

## Overview

Pocket Ledger is planned as an on-device pipeline: a user-approved import enters a local parser, normalized transaction candidates pass through validation and reconciliation, and accepted records are stored in an encrypted local ledger. The interface reads from that ledger to show combined and account-level spending.

## Planned components

- **Import boundary.** Starts with SMS selected through an explicit Android permission flow. Gmail and statement import remain open decisions rather than assumed dependencies.
- **Source adapters.** Convert supported alert formats into transaction candidates while preserving a safe reference to the source type and masked account.
- **Normalizer.** Produces a consistent record for amount, direction, merchant or counterparty, time, institution, account type, and masked identifier.
- **Validation and deduplication.** Drops declined alerts, detects repeated notifications, and flags incomplete or ambiguous records for review.
- **Reconciliation rules.** Separates real spending from refunds, transfers between owned accounts, and credit-card repayments.
- **Local ledger.** Stores transactions, accounts, categories, corrections, and import history on the device. The v0.3.6 prototype keeps this store on the device and deletes it on uninstall. Encryption and migration are still open, and backup is not implemented.
- **Views and corrections.** Presents combined spending and per-account filters, with manual entry and category correction.
- **User-controlled export.** The private prototype can export a CSV by year when the user asks for it. An encrypted portable backup is not implemented. Automatic cloud backup is not part of the design.

## Planned data flow

1. The user enables and starts an import.
2. A source adapter reads only the selected input and produces transaction candidates.
3. The normalizer maps candidates into a consistent local model.
4. Validation removes declines and duplicate alerts.
5. Reconciliation labels refunds, owned-account transfers, and card repayments before totals are calculated.
6. Ambiguous records wait for user review; accepted records enter the local ledger.
7. The UI derives combined and per-account views from the ledger.

## Trust boundary

- Financial content stays on the Android device.
- The core product does not require an account, analytics SDK, app-owned server, or cloud AI call.
- Imports are opt-in and source-specific.
- Corrections remain visible and attributable to the user.
- Synthetic or redacted fixtures are used for development.

## Open decisions

- Gmail or statement import, which are outside the SMS prototype
- Supported institutions and alert formats
- Local database encryption and migration
- Matching rules for partial refunds and multi-part transactions
- Encrypted backup and recovery, beyond the current user-started CSV export by year

## Held back

The application source, bank-specific parsing templates, reconciliation rules, test corpus, credentials, signing material, and every real financial record are private and are not in this repository.
