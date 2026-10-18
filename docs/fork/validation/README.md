# Feature validation record

These are software logs for feature commit `b3c607d`, recorded on 2026-10-18 in disposable Arch Linux containers. They do not contain real MSI acceptance results. The final source-file checksums matched the checkout before commit.

| Check | Result | Record |
| --- | --- | --- |
| `make lint` | Passed | [Lint](lint.log) |
| `make test` | 237 cases passed, including package building | [Regression suites](test.log) |
| `make test-mutations` | All 315 listed mutations caught, including F330 to F336 | [Mutations](mutations.log) |
| `make test-contract` | 10 sbctl cases and 13 Limine cases passed | [Contracts](contracts.log) |

The contract packages were sbctl 0.18-2, Limine 12.9.3-1, limine-mkinitcpio-hook 1.38.0-1.1 and util-linux 2.42.4-1, from Arch and Omarchy stable repositories. The MSI match in the contract is produced by synthetic DMI inside the sandbox, not by testing the user's motherboard.

[Source checksums](feature-manifest.sha256) cover the twelve files of the feature commit, before this handoff material was added. From the repository root:

```bash
sha256sum -c docs/fork/validation/feature-manifest.sha256
```

These logs use temporary fixture paths and synthetic data. They contain no host firmware export or private signing keys. A new clone should rerun the applicable checks before further changes are published.

Trailing whitespace was trimmed from the stored logs; messages and results are unchanged.
