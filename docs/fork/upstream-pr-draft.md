Title: Report sbctl firmware quirks in status

`omasecboot status` omits the known firmware quirks that sbctl already reports, including MSI's FQ0001. Show each match's ID, name, severity and guidance link, both before and after setup. State that these are hardware matches rather than a test of the current firmware policy: matches warn without changing the exit status, while a failed or unreadable diagnostic needs attention. Preserve silent `status --quiet` behaviour.

This implements the firmware-quirk reporting item in the maintenance plan, updates the status specification and C4 contract, and adds hermetic and real-tool tests with synthetic DMI. The change has no MSI hardware acceptance record; C10 keeps that evidence owed. The production change is confined to the read-only report.

Validation on the final files in disposable Arch containers:

- `make lint`: passed.
- `make test`: passed, 237 cases including package building.
- `make test-contract`: passed, 10 sbctl cases and 13 Limine cases with Omarchy stable packages (sbctl 0.18-2, Limine 12.9.3-1, limine-mkinitcpio-hook 1.38.0-1.1, util-linux 2.42.4-1).
- `make test-mutations`: passed, all 315 listed mutations caught, including the seven added entries.

The seven added mutation entries cover omitted reporting, invalid diagnostic shape, failure-to-attention handling, multiple JSON documents, control characters, command failure despite valid output, and empty fields. No hardware test is claimed.
