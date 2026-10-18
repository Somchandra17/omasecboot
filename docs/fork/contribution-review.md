# OmaSecBoot contribution review

Reviewed against checkout `bc72e33`, CONTRIBUTING.md, the design and upstream contracts, the field-testing and maintenance guides, the Omarchy integration proposal, and the local ykzird-secure-boot scripts. Work is on `feature/firmware-quirk-status`.

The goal is useful: establish evidence for MSI firmware and improve the diagnostics that help operators keep an installed Omarchy system healthy with Secure Boot enabled. The first contribution should extend the existing report and close a documented gap. A hardware-specific fork with new command families would be a much larger proposal than this project currently needs.

| Proposed work | Revised disposition |
| --- | --- |
| `doctor` and `safe` | Extend `status`. It already reports enrollment, boot files, settings, Windows integration, hooks, watchers and recovery actions. `status --quiet` is the existing automation interface. A clean report must not claim that firmware will boot the machine, enforce every signature or satisfy a game's anti-cheat. |
| MSI compatibility | Surface sbctl's existing firmware-quirk matches, including FQ0001, with its guidance and the limits of the match. Do not infer a workaround from the vendor name. This is the implemented first patch. |
| MSI B760 acceptance record | Follow the existing staged recorder on the actual Omarchy installation. Board, BIOS, CPU, GPU and Windows details in the proposal are user-supplied context, not verified acceptance results. No hardware record has been created. |
| Omarchy post-update audit | Keep verification through `status --quiet`. The repository already proposes an authorised step after `omarchy-update-status`; work with Omarchy on that integration. The ykzird user hook runs another signer through sudo and is not a drop-in implementation of this proposal. |
| Firmware-update preflight/postcheck | A possible later design proposal. Reuse the existing firmware backup and observed enrollment checks; define what a baseline records and how it behaves after snapshots. No automatic key restoration or invented successful checks. |
| Package-derived rebuild | Investigate separately. Cache extraction alone authenticates nothing. Verify the package's signature and intended version, account for the Limine tools deliberately retaining an older major, and define a failure path before changing the loader. The existing unauthenticated raw source is already an explicit threat-model limit. |
| Windows reboot | The package already ships a menu action that requests BootNext and reboots on success. Preserving a pending BootNext from another tool is a documented upstream decision still needed before that integration ships. A new command is lower priority. |
| Desktop warnings and encrypted export | Defer until there is a concrete requirement and a specification. Existing command output and the planned update audit provide the first notification path. Export must distinguish public firmware certificates from private signing keys. |

The proposed command output overstates what software can establish. In particular, an sbctl hardware match is not a live enforcement test; an enrolled certificate is not proof of successful boot; finding Windows Boot Manager is not proof that its loader is trusted; and BootNext is a request, not proof that Windows started or BitLocker stayed quiet.

“Secure Boot permanently on” is a goal for normal operation after setup. The current installer and the project's documented field-testing and recovery procedures can require it off. Level 1 explicitly starts with Secure Boot off and factory keys; it also modifies boot files, so it is not a read-only collection step. Do not try to run the Omarchy setup procedure on the present Fedora environment.

## Implemented patch

`status` reports each sbctl firmware quirk's ID, name, severity and link before or after setup. It explains that these are hardware matches, and does not turn an empty list into an assurance of enforcement. A match is an informational warning; an unreadable or failed diagnostic requires attention. Quiet mode remains silent. Signing and enrollment code are unchanged.

The patch adds hermetic cases, seven mutation entries (F330–F336), and a real sbctl contract using synthetic DMI inside the existing sandbox. Documentation, the specification, the changelog and the maintenance/integration lists follow the new behaviour. The tests do not constitute MSI hardware acceptance.

## Next contribution

After installing Omarchy, use the repository's field-testing guide and recorder, beginning with its prerequisites and Level 1. Publish only the sanitised records the sharing tool produces. Use that evidence to choose the next fix; a second firmware vendor and Windows Pro encryption are both evidence the maintainer still asks for.

## Sources

- [Contribution rules](https://github.com/peregrinus879/omasecboot/blob/main/CONTRIBUTING.md)
- [Maintenance gaps](https://github.com/peregrinus879/omasecboot/blob/main/docs/maintenance.md)
- [Field-testing procedure](https://github.com/peregrinus879/omasecboot/blob/main/docs/field-testing.md)
- [Omarchy integration proposal](https://github.com/peregrinus879/omasecboot/blob/main/docs/omarchy-integration.md)
- [sbctl 0.18 status JSON](https://github.com/Foxboron/sbctl/blob/0.18/cmd/sbctl/status.go)
- [sbctl quirk representation](https://github.com/Foxboron/sbctl/blob/0.18/quirks/quirks.go)
- [FQ0001 matching logic](https://github.com/Foxboron/sbctl/blob/0.18/quirks/fq0001.go)
- [FQ0001 operator guidance](https://github.com/Foxboron/sbctl/wiki/FQ0001)
- [MSI UKI report and maintainer response](https://github.com/omacom/omarchy/issues/12045)

The MSI UKI issue is a report on different hardware, with unresolved enrollment/enforcement questions. It does not establish a B760 failure or justify automatically disabling UKIs.

## Validation

All required checks passed on the final files in disposable Arch containers:

- `make lint`.
- `make test`: 237 cases, including the package build.
- `make test-mutations`: all 315 listed mutations caught, including F330–F336.
- `make test-contract`: 10 sbctl and 13 Limine cases, with sbctl 0.18-2, Limine 12.9.3-1, limine-mkinitcpio-hook 1.38.0-1.1 and util-linux 2.42.4-1 from the Arch and Omarchy stable repositories.
- `git diff --check`, and source-file checksums matching the tested copies.

The validation logs and source checksums are preserved under `docs/fork/validation/`; the patch itself is feature commit `b3c607d`. The first mutation attempt exhausted the default container's process limit; the complete passing run used an init process to reap children from the interruption tests. These are software results only. No hardware acceptance or game compatibility is claimed, and the host's boot files and firmware were not modified.
