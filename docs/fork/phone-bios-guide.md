# MSI Secure Boot: phone continuation guide

Keep this page on your phone while the computer is in BIOS. This is a personal continuation record for MSI PRO B760M-A WIFI (MS-7D99), BIOS E7D99IMS.AH0, observed on 18 October 2026. It is not a general installation procedure or a compatibility claim.

## Verified after manual import

- Secure Boot is **off**, Setup Mode is **off**. The imports persisted across reboot.
- Each local PK, KEK and db certificate matches by type, owner and SHA-256. Each complete firmware payload is also byte for byte identical to its staged ESL file.
- PK: 1254 bytes/1 entry; KEK: 5147 bytes/4 entries; db: 9709 bytes/7 entries, excluding efivarfs's four attribute bytes.
- Every original KEK/db entry remains. dbx, including its attributes, is byte for byte unchanged from the original pre-enrollment backup.
- Root `sudo omasecboot status` exited 0. The current UKI and primary loader signatures, configuration seal and watchers pass. This boot used the signed primary Limine path on the correct Linux EFI partition.
- The raw fallback and two older unsigned snapshot images require Secure Boot off.
- The installer USB is available. The user reports that Windows on its separate SSD is not using BitLocker.

**No Secure Boot-on boot or signature-enforcement test has passed yet.**

## Next: inspect MSI's execution policy

The last reported page showed **System Mode: User**, **Secure Boot: Disabled**, **Secure Boot Mode: Custom** and **Secure Boot Preset: Hardware/OS Comp...** (truncated), unchanged. The user also recalls **Maximum Security** being offered on the previous visit. Reconfirm it in the current menu. The FQ0001 hardware match does not inspect the current policy.

On the computer, check the boot files before entering BIOS:

```bash
sudo omasecboot status --quiet && systemctl reboot --firmware-setup
```

If the check fails, remain in Omarchy and run `sudo omasecboot status` for details.

Open **Security → Secure Boot**, leave Secure Boot **Disabled** and mode **Custom**, then open **Secure Boot Preset** and describe the full choices to the phone assistant. [Upstream FQ0001 guidance](https://github.com/Foxboron/sbctl/wiki/FQ0001) specifies:

- For **Secure Boot Preset**, select **Maximum Security** if that exact choice is offered.
- For the alternative **Image Execution Policy** menu, set **Option ROM**, **Removable Media** and **Fixed Media** to **Deny Execute**.

If the labels differ, report the choices before enabling. If selecting a preset proposes replacing, deleting or enrolling keys, cancel that operation and report it. The imported keys are already verified. Do not infer the policy from an unknown or truncated label, or write a private firmware variable to change it.

After confirming and correcting the actual policy, coordinate the final enable with the assistant. Preserve Custom mode and imported keys. Review the save-and-exit confirmation and save the changes; **Reset without saving** does not save changes. Boot the normal **Omarchy** entry through **Limine**, rather than an older unsigned snapshot or the raw **UEFI OS** fallback. If the boot is refused, disable Secure Boot in BIOS, save and start the normal entry again, preserving the keys and reporting the exact error.

After a successful boot, the local assistant must verify the live SecureBoot variable, enrollment and boot files again. An enabled indicator and a signed boot do not alone prove rejection of untrusted images; that remains a separate enforcement test.

## Observed import workflow: completed, do not repeat

Software enrollment failed on its first db write, including a retry that proved effective UID 0. Its underlying authenticated-write rejection remains unresolved; missing sudo does not explain it. KEK, db and dbx remained unchanged after that failure.

Complete public ESL files were then imported manually in this order:

| Order | Variable | File on Linux EFI partition |
| --- | --- | --- |
| 1 | db | `EFI\omasecboot-enroll\db.esl` |
| 2 | KEK | `EFI\omasecboot-enroll\KEK.esl` |
| 3 | PK, last | `EFI\omasecboot-enroll\PK.esl` |

Observed workflow: **Set New Var → No to factory defaults/load from file → Linux EFI filesystem → file → Public Key Certificate → confirm the matching variable and filename**. This BIOS groups EFI Signature Lists under **Public Key Certificate**. The Linux filesystem was fourth in that session, not a stable selection rule. Some summary rows were stale until revisited. No dbx operation, key deletion, factory-key enrollment or TPM clearing was reported. These files contain public certificates, not private signing keys.

Original reference backup: `/var/lib/omasecboot/firmware-backup/20261018T151648Z`. Do not replace it with current variables. MSI documents individual key operations in its [Intel 700 BIOS manual](https://download.msi.com/archive/mnu_exe/mb/Intel700BIOS.pdf), pages 30–31; the observed firmware calls the action **Set New Var**.

## Prompt for the phone assistant

```text
Continue from the completed imports; do not repeat enrollment. My local assistant is unavailable during BIOS. Guide me using the menu text I describe; photos are optional.

MSI PRO B760M-A WIFI (MS-7D99), BIOS E7D99IMS.AH0. Omarchy with Limine and a UKI, Windows on a separate SSD, no BitLocker according to me, installer USB available.

After manual db, KEK and PK imports, the local assistant checked the rebooted machine as root. SecureBoot=0 and SetupMode=0. All three local certificates match, each complete firmware payload is byte-identical to its staged ESL, every original KEK/db entry remains and dbx exactly matches the ORIGINAL backup. The UKI and primary loader signatures, configuration seal, watchers and actual primary boot path pass. No enrollment repair is needed. The raw fallback and two old unsigned snapshots require Secure Boot off.

The remaining BIOS step is FQ0001 execution policy, then final enable. Last reported page: System Mode User, Secure Boot Disabled, Custom mode, Secure Boot Preset truncated as Hardware/OS Comp..., unchanged. First help me open Secure Boot Preset and inspect the full choices while Secure Boot stays off. Upstream https://github.com/Foxboron/sbctl/wiki/FQ0001 specifies Maximum Security for that menu, or Deny Execute for Option ROM, Removable Media and Fixed Media if Image Execution Policy is offered instead. Do not guess at different labels. If a preset change proposes replacing/deleting/enrolling keys, cancel and report it.

Once the actual policy is confirmed and corrected, help me enable Secure Boot while preserving Custom mode and imported keys, review the save-and-exit confirmation, and boot the normal Omarchy entry through Limine. If boot fails, disable Secure Boot again, preserve keys, boot normally and report the exact error. Do not choose Reset without saving to save changes. Do not rerun enrollment, force writes, clear keys, restore factory keys or clear TPM. No Secure Boot-on boot or signature-enforcement test has passed yet. A signed boot alone does not prove untrusted images are rejected. After returning, the local assistant must verify the live state again.

Full guide:
https://github.com/Somchandra17/omasecboot/blob/feature/firmware-quirk-status/docs/fork/phone-bios-guide.md
```
