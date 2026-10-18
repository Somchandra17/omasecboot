# Field report: MSI PRO B760M-A WIFI — root enrollment failure, verified manual import and successful boot

Observed 18 October 2026. Prepared for upstream maintainer review. This is a partial field run with a firmware-specific enrollment deviation, not a completed hardware acceptance level.

## Machine and software

- Board: MSI PRO B760M-A WIFI (MS-7D99).
- Firmware: American Megatrends International, LLC.; DMI A.H0; BIOS image E7D99IMS.AH0; build date 25 August 2026.
- Omarchy 4.0.5-1, linux-omarchy 7.2.8-9, Limine 12.9.3-1, limine-mkinitcpio-hook 1.40.0-2, sbctl 0.18-2, OmaSecBoot 0.1.1-1.
- Production feature tested: commit b3c607d25b23c71fcd1d6914e6e14d6a87d2bc62, based on upstream bc72e33. Later commits on the fork add handoff/validation notes; the installed production files match the checkout. The latest recorder row identifies fork commit b0aae64 and reports `Evidence: counts`.
- Windows is on a separate SSD. The operator reports no BitLocker; this was not independently measured inside Windows. Installer USB available.

## Result

Reached a normal signed Omarchy boot with Secure Boot on after manual firmware imports. The operator also reports that Windows started from the managed Windows entry in Limine. Automated enrollment did not succeed, and no claim is made that this machine completed all Level 1/2 rows or stages 0–7.

## What differed

### 1. Software enrollment was rejected despite verified root

After deleting only the PK and saving/rebooting, live variables read `SetupMode=1`, `SecureBoot=0`, `AuditMode=0`. KEK and db retained their existing entries. BIOS mode was Custom and Secure Boot was disabled.

Both `2-enroll` and `2-enroll-root` exited 1 at the first db write. The second transcript explicitly contains:

```text
uid=0(root) gid=0(root) groups=0(root)
Effective UID: 0
```

The failure was:

```text
sbctl requires root to run: couldn't sync keys: couldn't write efi variable: write /sys/firmware/efi/efivars/db-d719b2cb-3d3a-4596-a3bc-dad00e67656f: permission denied
```

The `requires root` prefix does not establish missing sudo: effective UID was proved. The underlying write rejection remains unresolved. Do not infer its cause from the motherboard name, Custom mode, or the Provision Factory Default keys setting alone.

Read-only root comparisons after failure found KEK, db and dbx byte for byte unchanged from the original pre-enrollment backup. No local key write succeeded. No forced write, blanket key deletion, factory-key restoration or TPM clearing was performed.

### 2. Manual import of the verified complete public ESL files succeeded

The helper exported complete public ESL plans using sbctl and checked them against OmaSecBoot's append plan. Existing KEK/db entries were proved present in the plans before the files were staged on the Linux ESP. No private signing keys were copied.

The operator imported, in order:

1. `EFI/omasecboot-enroll/db.esl` into db.
2. `EFI/omasecboot-enroll/KEK.esl` into KEK.
3. `EFI/omasecboot-enroll/PK.esl` into PK, last.

Reported BIOS route on this firmware:

**Security → Secure Boot → Key Management → variable → Set New Var → No to factory defaults/load from file → Linux EFI filesystem → matching file → Public Key Certificate → matching variable/file confirmation.**

This MSI BIOS accepts the complete EFI Signature Lists through the label **Public Key Certificate**. The Linux ESP was fourth in that session; filesystem ordering is not a reliable general identifier. Some summary rows were stale until revisited. dbx was left alone.

After saving and returning to Linux, independent root checks found:

| Check | Result |
| --- | --- |
| PK | 1254 payload bytes, 1 entry; matching local PK certificate |
| KEK | 5147 payload bytes, 4 entries; matching local KEK certificate |
| db | 9709 payload bytes, 7 entries; matching local db certificate |
| Full payload comparison | Each of PK, KEK and db byte-identical to its prepared complete ESL file |
| Original KEK trust | All 3 original entries retained by signature type and certificate digest |
| Original db trust | All 6 original entries retained by signature type and certificate digest |
| dbx | Byte-identical, including efivarfs attributes, to the ORIGINAL pre-enrollment backup |

The backup's own checksums and ownership passed; current variables were not substituted for the original reference. Counts and BIOS labels were not used as proof of enrollment.

### 3. FQ0001 and the final enable

The BIOS save confirmation was photographed listing:

```text
Secure Boot: [Disabled] -> [Enabled]
Secure Boot Preset: [Hardware/OS Compatibility] -> [Maximum Security]
```

The preset selection follows sbctl's FQ0001 guidance. After reboot, root reads found `SecureBoot=1`, `SetupMode=0`, `AuditMode=0`, `DeployedMode=0`; `bootctl status` also reported enabled (user).

The current boot entry is the active primary Limine entry, first in BootOrder, on the partition mounted at `/boot`. The running UKI is on that same partition. The current UKI and primary loader signatures, configuration seal and watchers pass. All three imported payloads, preserved original KEK/db entries and unchanged dbx were checked again after enabling. The raw fallback still equals the packaged raw loader.

FQ0001 remains reported by sbctl after selecting Maximum Security. Its hardware/date match does not test the current preset. No deliberate rejection test of an untrusted EFI image was run, so a signed boot and the chosen preset are not presented as independent proof of signature enforcement.

### 4. Windows menu

Windows Boot Manager existed as an active firmware entry but the managed Limine Windows entry was not enabled. `omasecboot windows setup` added that entry and re-sealed/signed the loader. Root Windows status and full status passed afterwards. The operator then reported a successful Windows boot from the menu. No Windows-side measurement or BitLocker acceptance test is claimed. The separate `omasecboot windows bootnext` command was not tested.

## Other difference

The initial `1-setup` exited 1 because the combined question about having the Windows recovery key OR having no encrypted Windows was answered No: the operator meant that BitLocker was not in use. Clarifying the question and answering Yes on the next run allowed setup to complete. This suggests a wording improvement without weakening its existing safety condition.

## Completed, failed and unrun

- **Passed:** boot-file preparation; normal Secure Boot-off boot; root status and signature/seal/watcher checks; manual-import comparisons against all three local certificates and the original backup; normal Secure Boot-on Omarchy boot; latest recorded root status.
- **Failed:** both automated db enrollment attempts, including the verified-root attempt.
- **Operator-reported:** BIOS import dialogue; the manual import order; Windows boot from the managed Limine entry; Windows encryption off. The final policy/enable save dialogue is also supported by a photograph; it is not attached here because it contains the surrounding room.
- **Unrun:** kernel and Limine reinstall/update acceptance rows; signed snapshot boot/restore; deliberate rejection of an untrusted EFI image; standalone BootNext command; removal/factory-key restoration and way-back verification; destructive drills; encrypted-Windows testing. Two old snapshot images remain unsigned and require Secure Boot off.
- **Separate limitation:** Linux lockdown read `[none]`. This report does not claim kernel-module signature enforcement or game/anti-cheat compatibility.

The working enrollment has been preserved; no repeat enrollment or key clearing is needed to report these observations.

## Sanitised records

[Download the sanitised recorder archive](omasecboot-records.tgz). It contains eleven recorder rows: `0-before-install`, `0-baseline`, `1-setup`, `1-setup-again`, `1-status`, `1-status-reboot`, `2-before-pk-delete`, `2-after-pk-delete`, `2-enroll`, `2-enroll-root`, and `2-status-msi-current`.

The latest row ran `omasecboot status` as root, recorded `Evidence: counts`, recorded `SecureBoot=1`, and its command exit was 0. Each row's command exit is assessed separately from the recorder's own exit and its `Evidence` header.

Copies were generated with `tests/acceptance-share.sh`. Residual login names in pkexec journal entries were additionally redacted in the copies; archive ownership is numeric. Source records remain private and unchanged. Supplemental inspection logs support the manual-import comparisons and Windows-menu setup; they are not recorder rows and are not included as raw files in the archive. No firmware backup payload, private key or BIOS room photograph is attached.

## Contribution requested

Please assess the root db-write rejection and this MSI BIOS's manual ESL-import behaviour. The existing firmware-quirk reporting PR #3 addresses visibility only; it does not fix enrollment. These findings can support a separate MSI recovery/enrollment documentation change and clearer permission-failure guidance once the maintainer agrees on the supported route. No automatic MSI workaround or enrollment retry is proposed in this report.

Archive SHA-256: `ae990c2cb3c493e512d3dca48cd49507253ebc70969d91e7edf0118d71843811`.
