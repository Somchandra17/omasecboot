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

The user asked for this handoff so an agent can resume without needing the old conversation. Continue useful repository work autonomously within the authorised scope. Detect the current OS and boot state first: Fedora was the last observed host; a new clone may be on Omarchy. Do not assume that Omarchy or OmaSecBoot has been installed or set up.

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

- [ ] Confirm which OS is now running and that this branch is checked out. Inspect repository state before editing; preserve any new user changes.
- [ ] If Omarchy is not installed, help the user follow its current official installation guidance. Installation must target the intended Linux SSD; full-disk installation erases the selected drive. The installer requires Secure Boot off. Leave the TPM and firmware keys alone during this preparation.
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
