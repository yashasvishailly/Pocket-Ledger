# Pocket Ledger, planned architecture

## Status

The private source reviewed on 22 October 2026 identifies v0.4.1. It implements
on-device SMS debit import, account and category corrections, an encrypted local
ledger, CSV export, and passphrase-encrypted backup and restore. This document's
component layout and broader reconciliation remain design direction.

Test builds send usage and crash reports only when a nonempty reporting key is
configured and the first-launch notice has been accepted. The reviewed v0.4.1
source has no in-app off switch after acceptance. Reports include approximate location from
the reporting provider's network metadata. Amounts, merchants, account details,
and SMS text are excluded from those reports.

## Overview

Pocket Ledger is planned as an on-device pipeline: a user-approved import enters a local parser, normalized transaction candidates pass through validation and reconciliation, and accepted records are stored in an encrypted local ledger. The interface reads from that ledger to show combined and account-level spending.

## Planned components

- **Import boundary.** Starts with SMS selected through an explicit Android permission flow. Gmail and statement import remain open decisions rather than assumed dependencies.
- **Source adapters.** Convert supported alert formats into transaction candidates while preserving a safe reference to the source type and masked account.
- **Normalizer.** Produces a consistent record for amount, direction, merchant or counterparty, time, institution, account type, and masked identifier.
- **Validation and deduplication.** Drops declined alerts, detects repeated notifications, and flags incomplete or ambiguous records for review.
- **Reconciliation rules.** Separates real spending from refunds, transfers between owned accounts, and credit-card repayments.
- **Local ledger.** Stores transactions, accounts, categories, corrections, and import history on the device. The reviewed v0.4.1 source encrypts this store with an Android Keystore key. Uninstall removes it; an exported encrypted backup and its passphrase are needed for recovery.
- **Views and corrections.** Presents combined spending and per-account filters, with manual entry and category correction.
- **User-controlled export.** The private prototype can export a CSV by year when the user asks for it. The reviewed source also implements a passphrase-encrypted portable backup and restore. Automatic cloud backup is not part of the design.

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
- Financial processing requires no account or cloud AI call. Test reporting sends usage and crash events to an external provider only when a nonempty reporting key is configured and the first-launch notice has been accepted. The reviewed v0.4.1 source has no in-app off switch after acceptance; this is not a claim that an unconfigured build sends reports.
- Imports are opt-in and source-specific.
- Corrections remain visible and attributable to the user.
- Synthetic or redacted fixtures are used for development.

## Open decisions

- Gmail or statement import, which are outside the SMS prototype
- Supported institutions and alert formats
- Matching rules for partial refunds and multi-part transactions
- Device checks for backup recovery and future storage migrations

## Held back

The application source, bank-specific parsing templates, reconciliation rules, test corpus, credentials, signing material, and every real financial record are private and are not in this repository.
