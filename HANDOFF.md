# Fork handoff

Updated 2026-10-18. Read this together with [CONTRIBUTING.md](CONTRIBUTING.md) before continuing. This is a fork's working record, not a hardware acceptance report or an installation script.

## Restore the work after reinstalling

```bash
git clone --branch feature/firmware-quirk-status https://github.com/Somchandra17/omasecboot.git
cd omasecboot
git status --short
git log -3 --oneline
```

The work is on `feature/firmware-quirk-status`, not the fork's `main` branch. GitHub authentication and the local Git author settings do not transfer with a clone. Use the user's GitHub identity when needed; this checkout used `Som` and `85082756+Somchandra17@users.noreply.github.com`.

The user asked for this handoff so an agent can resume without needing the old conversation. Continue useful repository work autonomously within the authorised scope. Detect the current OS and boot state first: Omarchy was observed on 2026-10-18 after the reinstall. Do not assume that OmaSecBoot has been installed or set up.

## Resumed on Omarchy, 2026-10-18

The user asked to set up Secure Boot on this machine while committing the work to their fork. The existing clone is on `feature/firmware-quirk-status`; the feature-file checksums match the preserved validation manifest.

Read-only checks found Omarchy 4.0.5-1, the running and installed `linux-omarchy` 7.2.8-9, Limine 12.9.3-1 and `limine-mkinitcpio-hook` 1.40.0-2. `bootctl status` identifies Limine and the running UKI. Secure Boot and Setup Mode both read 0. DMI identifies the MSI PRO B760M-A WIFI (MS-7D99), BIOS A.H0. This confirms the observed identity, not signature enforcement or hardware acceptance.

The Linux ESP is vfat, mounted read/write with `fmask=0077,dmask=0077`, with about 1.3 GiB available. Root-level inspection found one UKI of about 252 MiB, so two more fit, and the fallback loader matches the packaged raw Limine byte for byte. The firmware lists Limine, Windows Boot Manager and a fallback entry named `UEFI OS`; that entry's presence does not establish that it has been boot-tested. No pending packages were listed by `pacman -Qu` against the local databases; a completed Omarchy update and subsequent reboot still need confirmation. Setup Mode being off does not prove that the keys are the factory keys.

At the initial check, neither `sbctl` nor OmaSecBoot was installed. `sbctl` 0.18-2 and ShellCheck 0.11.0-158 have since been installed through pacman. Installing sbctl ran the normal initcpio hook, which rebuilt the UKI and updated `limine.conf`; the hook explicitly skipped signing because no signing keys exist. Root-level inspection found neither `/var/lib/sbctl` nor `/var/lib/omasecboot`. OmaSecBoot itself is not installed or set up, and no firmware variables have been changed.

The actual machine's `sbctl status --json` reports FQ0001, severity `CRITICAL`, by its date-matching method, and recognises Microsoft and the built-in PK, KEK and db. The match does not inspect the current image execution policy. Review [sbctl's FQ0001 guidance](https://github.com/Foxboron/sbctl/wiki/FQ0001) with the firmware menu in view before enabling Secure Boot; no policy correction or enforcement test has been performed.

The package built successfully and all 237 regression cases passed. Lint subsequently passed once ShellCheck was installed. Both real-tool contract suites passed with `CONTRACT_REQUIRED=1`: 10 sbctl cases and 13 Limine cases against the installed Limine 12.9.3-1, hook package 1.40.0-2, sbctl 0.18-2 and util-linux 2.42.4-1. This checks the newer hook package against the suite; it does not update every source-reviewed contract or establish hardware acceptance.

The user confirmed this is a fresh Omarchy installation, that the same installer ISO remains on the USB, and that Windows is not using BitLocker. Treat the Windows encryption statement as user-reported, not a Windows measurement. The clock reports NTP synchronised. The current firmware exposes its built-in PK, KEK and db as recognised by sbctl; the key-menu wording and the ability to delete only the PK still need inspection before Level 2.

### Live boot-file preparation completed

OmaSecBoot 0.1.1-1 from this branch is now installed and set up. Signing keys were created, the current UKI and primary Limine loader are signed, the loader is sealed over the current configuration, and both watchers are active. The raw fallback is preserved. The firmware's existing keys were backed up under `/var/lib/omasecboot/firmware-backup/20261018T151648Z`; no firmware key has been deleted or enrolled, and Secure Boot remains off. A recovery snapshot named `OmaSecBoot signed boot files; Secure Boot off` was created after the boot files were prepared.

Private recorder rows are in the ignored `acceptance-records/` directory: `0-before-install`, `0-baseline`, `1-setup`, `1-setup-again` and `1-status`. Their headers read `Evidence: counts`. The initial setup exited 1 after the user selected No at the combined recovery-key-or-no-encrypted-Windows question; the user explained that they chose No because Windows was not encrypted. The boot files were already prepared. After clarification, `1-setup-again` exited 0 and `1-status` exited 0. The report verifies the signatures, seal, raw fallback and watchers, and reports FQ0001 and unenrolled keys as expected. These are partial installation records, not a completed Level 1 run or a Secure Boot boot result. No raw records or signing keys have been committed.

The normal reboot and recorded `1-status-reboot` check subsequently passed. The user's first PK deletion did not persist; a subsequent deletion did: `SetupMode=1`, `SecureBoot=0`, `AuditMode=0`, with the original KEK and db still present. Both `2-enroll` and `2-enroll-root` failed at the first db write with permission denied; the latter explicitly recorded `uid=0(root)` and effective UID 0. The misleading "requires root" prefix comes from sbctl's general permission-error handler, not evidence that sudo was omitted. Root-level comparisons after the first failure found KEK, db and dbx byte for byte unchanged from the backup. No key write succeeded in either attempt.

### Manual import verified after reboot

The user completed the manual imports in the BIOS, db then KEK then PK last. The reported workflow was **Set New Var → No to factory defaults → Linux EFI filesystem → file → Public Key Certificate → confirm the matching variable and filename**. This BIOS accepted the complete ESL files through **Public Key Certificate**. The Linux EFI partition happened to be the fourth filesystem entry; do not use that position as a general rule. Some summary counts were stale until revisited. No dbx operation, key deletion, factory-key enrollment or TPM clearing was reported.

After the reboot at 21:49 local time on 18 October, an inspection explicitly printed `uid=0(root)` and ran `sudo omasecboot status`, which exited 0. `SecureBoot=0`, `SetupMode=0`, `AuditMode=0` and `DeployedMode=0`. Each enrolled local PK, KEK and db certificate matched by type, owner and SHA-256. All three firmware payloads were also byte for byte identical to their staged complete ESL files: PK 1254 bytes/1 entry, KEK 5147 bytes/4 entries, db 9709 bytes/7 entries.

The original backup at `/var/lib/omasecboot/firmware-backup/20261018T151648Z` passed its checksum and ownership checks. All three original KEK entries and all six original db entries remain present by signature type and certificate digest. The live dbx, including its efivarfs attributes, is byte for byte identical to that original backup; both SHA-256 values were `d85c43ea8bacfb6830070a5802fbabb81235f3a7339920ed2aae6f50f200c96d`. No new backup was substituted for that reference.

The current UKI and primary loader signatures, configuration seal and both watchers pass. `BootCurrent` is the active Limine entry, first in `BootOrder`, pointing to the signed primary loader on the partition mounted at `/boot`; the running UKI is on that same partition. The raw fallback remains byte for byte identical to the packaged loader. Two snapshot images predate setup and remain unsigned. Private output is in ignored `acceptance-records/2-manual-import-verification.log`; this is a supplemental inspection log, not a completed recorder row or hardware acceptance level.

**Immediate next step:** inspect the actual **Secure Boot Preset** choices with the [phone guide](docs/fork/phone-bios-guide.md) available. The last user-reported value was truncated as **Hardware/OS Comp...**, with Custom mode and Secure Boot disabled. The user recalls **Maximum Security** being offered on the previous visit. Reconfirm and select that option in the actual menu before enabling. The policy has not yet been corrected or proved safe. Upstream FQ0001 guidance calls for **Maximum Security** when that preset menu is offered, or **Deny Execute** for Option ROM, Removable Media and Fixed Media when Image Execution Policy is offered. Preserve imported keys and keep Secure Boot off until the policy is resolved. No Secure Boot-on boot or signature-enforcement test has passed. The kernel/Limine reinstall rows and the way back remain unrun. Do not rerun enrollment, force writes, delete keys, restore factory keys or clear the TPM.

The user's priority is getting Secure Boot working; contribution work accompanies that goal. Avoid further documentation-only checkpoints or repeated software runs while a real-machine step is ready. A concrete later contribution is clearer wording for the encryption acknowledgement: the current combined question caused the initial cancellation. Keep its existing safety condition; no wording patch has been made yet.

## User's goal and hardware context

Contribute small, reviewable improvements upstream while moving to Omarchy with Windows on a separate SSD and Secure Boot enabled during normal use. The fork is a development branch, not a plan to maintain a permanent alternative signing stack.

The following were supplied by the user and have not been verified by hardware acceptance:

| Item | Reported value |
| --- | --- |
| Board | MSI PRO B760M-A WIFI, MS-7D99 |
| Firmware | AMI / MSI, A.H0 |
| CPU | Intel Core i5-14600K |
| GPU | NVIDIA RTX 4070 SUPER |
| Windows | Windows 11 Pro, used for gaming including Battlefield 6 |
| Storage | Separate Windows and Linux SSDs; intended Omarchy target is 512 GB |

Do not turn these details into a compatibility claim. BitLocker state, key-menu behaviour, actual firmware version, signature enforcement and successful UKI boot remain unverified. The current patch does not establish game compatibility.

## Completed work

- [x] Read the contribution rules, specification and maintenance/integration plans; inspect the local ykzird scripts.
- [x] Rework the original proposal into a smaller first contribution. See [the contribution review](docs/fork/contribution-review.md).
- [x] Implement firmware-quirk reporting in `status`, feature commit `b3c607d` (`Report sbctl firmware quirks in status`).
- [x] Add tests, seven mutation entries and a real sbctl contract with synthetic DMI; update the specification, contracts, changelog and maintenance/integration pages.
- [x] Verify the feature in disposable Arch containers: lint, 237 regression cases, all 315 mutations, 10 sbctl contract cases and 13 Limine contract cases. See [validation records](docs/fork/validation/README.md).
- [x] Prepare an [upstream PR description](docs/fork/upstream-pr-draft.md). No upstream PR or field report has been submitted.

The production change is confined to `lib/status.sh`. It prints sbctl's quirk ID, name, severity and guidance link before or after setup. Matches warn without changing exit status because they are hardware matches, not tests of current firmware policy. A failed or unreadable diagnostic requires attention. `status --quiet` stays silent. Signing and enrollment logic are unchanged.

The comparison project is [ykzird-secure-boot](https://github.com/ykzird/ykzird-secure-boot). The old host had a separate checkout; it is not included here. Clone it again only if needed for source comparison, and do not install its signer alongside OmaSecBoot.

There is no `doctor` command or flag, and no `safe` command. The user asked about `doctor`; the decision so far was to extend the existing `status` report rather than add another diagnostic interface.

The feature commit is separate from this fork-only handoff material so it can be proposed upstream independently. The user authorised committing and pushing to their fork. Opening an upstream PR or posting a hardware report remains a separate publication step.

## Next tasks, in order

- [x] Confirm which OS is now running and that this branch is checked out. The resumed checkout was clean, on the feature branch, on Omarchy 4.0.5.
- [x] Confirm Omarchy is installed. No installation or drive erasure is needed on the observed host.
- [ ] Once installed, confirm normal Omarchy boot, update it and restart with `systemctl reboot`, keeping Secure Boot off. Do not infer a functioning boot chain from the software tests in this repository.
- [ ] Read [the field-testing prerequisites](docs/field-testing.md#before-you-start) and [the recovery guide](docs/recovery.md) with the user. Establish recovery media, the Windows recovery key, ESP space, the current factory/custom key state and the firmware's key-menu wording before any setup. If the prerequisites do not apply, report that rather than changing keys to force them to apply.
- [ ] Run the documented Level 1 procedure on the actual MSI machine, with the recorder at every prescribed row. Level 1 changes boot files and settings while Secure Boot remains off; it is not a read-only diagnostic. Coordinate the real-machine steps with the user; do not run them unattended merely because this file lists them.
- [ ] Inspect the MSI firmware warning and raw sbctl output; record the actual board, BIOS and firmware behaviour. A quirk match may remain after the image execution policy is corrected, and no match proves nothing about enforcement.
- [ ] Follow the guide's way back and sanitise records with `tests/acceptance-share.sh`. Inspect the copies before sharing. Raw acceptance records are deliberately ignored by Git; never publish recovery keys, private signing keys or unsanitised machine records.
- [ ] If the prerequisites and Level 1 results permit, coordinate Level 2 with the user to enroll keys and test Secure Boot. Follow the existing guide one firmware step at a time; do not improvise automatic PK deletion, factory-key restoration or TPM clearing.
- [ ] Prepare a hardware field report from real observations, distinguishing completed, failed and unrun rows. Ask for the user's instruction before posting it publicly.
- [ ] Use observed failures to choose the next code contribution. Update this checklist and keep the fork backed up as work progresses.

## Contribution priorities after hardware evidence

1. Review the existing feature for an upstream PR using the prepared description. Keep fork-only notes out of the proposed upstream change; the feature is one separate commit.
2. Coordinate Omarchy's post-update verification through `status --quiet`, following [the existing integration proposal](docs/omarchy-integration.md). It is proposed, not currently installed by this patch. Do not install a second signer or overwrite a user's post-update hook.
3. Consider the documented pending BootNext preservation work before adding a new Windows reboot command. The project already ships a reboot-to-Windows menu action.
4. Treat firmware-update preflight/postcheck and package-derived loader rebuilding as separate specification proposals. Package-cache extraction alone authenticates nothing, and selecting the newest package can conflict with the Limine tools deliberately retaining an older major.
5. Revisit desktop notifications and encrypted recovery export only with a concrete requirement. Keep public certificates distinct from private signing keys.

Never disable UKIs merely because the board is MSI. Do not add Battlefield-specific code, a second pacman signer, automatic firmware-key tricks, or a blanket assurance that it is safe to reboot. The contribution guide, specification, contracts and actual hardware records decide what can be claimed.

## Verification when continuing

Follow the contribution guide: run `make lint` and `make test` after changes; run `make test-mutations` before publishing code or test changes, and `make test-contract` when calls to upstream tools or their stubs change. A new or changed `limine.conf` reader also requires the replay suite. A skipped contract suite is not a pass of its cases.

On a non-Arch host, use a disposable Arch container for package-dependent checks. Give it an init process to reap children: the interruption tests otherwise leave processes behind, and the first mutation attempt exhausted the default container's process limit. The complete passing feature run used an init process and 16 workers. Contract suites need bubblewrap's nested mount/namespace support, as the repository's contract CI documents. Never mount the host's keys, ESP or firmware variables into a test container.

The earlier temporary containers were removed. The committed records preserve the results; a fresh machine must recreate the test environment. Hardware testing is still owed even when every software check passes.
