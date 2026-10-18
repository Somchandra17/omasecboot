# MSI key import: phone guide

Open this page on your phone before restarting the computer. This is a personal continuation guide for the MSI PRO B760M-A WIFI (MS-7D99), BIOS A.H0, observed on 18 October 2026. The BIOS import has not yet been completed or verified. It is not an upstream installation procedure or a compatibility claim.

## Current position

- Omarchy boots normally through the signed UKI and the signed Limine loader. The loader's configuration seal and the watchers pass the installed tool's checks.
- Secure Boot is disabled, Setup Mode is enabled and the Platform Key is absent. Custom mode was selected in the BIOS.
- The original three KEK entries, six db entries and dbx remain. Root-level comparisons found KEK, db and dbx byte for byte unchanged from the backup after the failed write.
- `omasecboot setup` failed at its first db write, including a recorded retry that printed `uid=0(root)` and effective UID 0. It has not enrolled the local certificates. The cause of the authenticated-write rejection remains unresolved.
- The user has the Omarchy installer USB and reports that Windows is not using BitLocker.
- Public signature-list files are prepared on the Linux EFI partition under `EFI/omasecboot-enroll/`. They were exported by sbctl and checked against OmaSecBoot's append plan. They contain the existing entries plus the local certificates. No private signing keys were copied.

## Before entering BIOS

Keep this page available on the phone. On the computer:

```bash
sudo omasecboot status --quiet && systemctl reboot --firmware-setup
```

If the report fails, remain in Omarchy and run `sudo omasecboot status` to see why.

## Import the prepared lists

Keep **Secure Boot disabled** and **Secure Boot Mode set to Custom**.

Open **Security → Secure Boot → Key Management**. The photographed BIOS menu offers **Set New Var** when a key database is opened.

Use **Set New Var** with each complete file below, in this order. The files already include the existing certificates, so this preserves their trust when the correct complete file is imported.

| Order | BIOS entry | File on the Linux EFI partition | Expected key count afterwards |
| --- | --- | --- | --- |
| 1 | Authorized Signatures (db) | `EFI\omasecboot-enroll\db.esl` | 7 |
| 2 | Key Exchange Keys (KEK) | `EFI\omasecboot-enroll\KEK.esl` | 4 |
| 3 | Platform Key (PK), last | `EFI\omasecboot-enroll\PK.esl` | 1 |

Select the option that loads a file. If the prompt specifically asks whether to load factory defaults, decline that choice and use the file-loading option. If a format is requested, these files are **EFI Signature Lists**, without efivarfs's four attribute bytes or an authenticated-update wrapper.

The exact file-picker dialogue has not been observed. If it does not offer the file-loading route, cannot find the files, rejects the format, or the counts differ, cancel the pending action with Escape. Leave Secure Boot off and return to Omarchy. Describe the last dialogue in the chat; no photo is required. Keep any successful imports in place for inspection rather than deleting variables to start again.

Leave **Forbidden Signatures (dbx)** unchanged. Do not use **Delete all Secure Boot variables** or **Enroll all Factory Default keys**. Finish by saving and booting Omarchy with Secure Boot still disabled.

The counts are a visual check, not proof of preserved certificates. After returning, the actual variables must be compared with the original backup and local certificates before Secure Boot is enabled.

## After returning to Omarchy

Tell the local assistant which imports succeeded. It can record the state and verify all three local certificates, every original KEK/db entry and unchanged dbx. A read-only report is also available:

```bash
sudo omasecboot status
```

Do not enable Secure Boot until enrollment and the boot files have been verified. Before the final enable, inspect MSI's image-execution policy: sbctl reports FQ0001, and a hardware match does not inspect the current setting. Follow [sbctl's FQ0001 guidance](https://github.com/Foxboron/sbctl/wiki/FQ0001) for the actual menu offered. No firmware enforcement test or Secure Boot-on boot has been completed.

MSI documents the individual key-management operations in its [Intel 700 BIOS manual](https://download.msi.com/archive/mnu_exe/mb/Intel700BIOS.pdf), pages 30–31. The photographed firmware labels the operation **Set New Var**.

## Prompt for a new ChatGPT conversation on your phone

Copy the following into a new phone chat:

```text
Help me continue Secure Boot setup from my phone while my PC is in BIOS. My local assistant is unavailable during BIOS, so do not assume you can run commands or inspect the PC. Guide me using the menu text I describe; photos are optional.

Hardware: MSI PRO B760M-A WIFI (MS-7D99), BIOS A.H0. Omarchy with Limine and a UKI; Windows is on a separate SSD. I have the installer USB and am not using BitLocker.

Completed: signing keys exist; the UKI and primary Limine loader are signed; the configuration is sealed; watchers are active. A normal reboot and the subsequent status check passed. The raw fallback and a recovery snapshot exist.

Current firmware: Secure Boot disabled, Custom mode, SetupMode=1, AuditMode=0, PK absent. Three original KEK entries, six original db entries and dbx are preserved. Provision Factory Default keys was shown as Enabled; do not assume this caused the failure.

Software enrollment failed at the first db write with permission denied, twice. The retry proved uid=0(root), effective UID 0. The error is not explained by missing sudo. KEK, db and dbx were compared afterwards and still matched the backup byte for byte. The underlying rejection is unresolved. Do not force enrollment, clear every key or clear the TPM.

Prepared public files on the Linux EFI partition:
EFI/omasecboot-enroll/db.esl
EFI/omasecboot-enroll/KEK.esl
EFI/omasecboot-enroll/PK.esl
They contain the complete existing lists plus my certificates, were checked against the append plan, and contain no private keys.

BIOS path: Security > Secure Boot > Key Management. Opening Authorized Signatures (db) shows Details, Save To File, Set New Var, Append Key and Delete Var. The planned manual fallback is Set New Var from the prepared complete EFI Signature List files: db first (expected 7 keys), KEK next (4), PK last (1). Leave dbx unchanged and Secure Boot disabled. The file-picker dialogue has not been observed, so help me identify the file-loading option without guessing at ambiguous confirmations. If an import fails, leave completed imports intact and return to Linux for inspection.

Afterwards, the local assistant must verify enrollment, preserved KEK/db certificates and unchanged dbx before enabling Secure Boot. MSI FQ0001 is reported; review its actual image-execution policy before the final enable. No Secure Boot-on boot or signature-enforcement test has passed yet.

The full phone guide is at:
https://github.com/Somchandra17/omasecboot/blob/feature/firmware-quirk-status/docs/fork/phone-bios-guide.md
```
