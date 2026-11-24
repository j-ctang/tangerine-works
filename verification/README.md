# Verification catalog

Use this catalog when adding CI to a product repo. The commands below are suggested entry points for scripts implemented in that repo. Each repo supplies its build commands, supported targets, fixtures, and user journeys. Use its existing test runner where possible and call the same entry points locally and in CI.

## Pull request checks

| Suggested command | What passes | When to run |
| --- | --- | --- |
| `verify-static` | Applicable formatting, lint, compiler or type checks pass. Architecture rules reject forbidden dependencies where the repo defines those boundaries. | Every code PR. |
| `verify-tests` | Unit and integration tests pass against isolated fixtures. Assert outputs and state changes callers can observe. | Every code PR. |
| `verify-workflow <target> <scenario>` | Start with known test data, wait for readiness, exercise the normal entry point, and assert the expected outcome. Read persisted data back when applicable. | On PRs affecting the workflow or its dependencies. |
| `verify-regression <base> <candidate> <scenario>` | Run the same check against both revisions. The base fails on the expected assertion and the candidate passes. For a new feature, a deliberate break can establish that the check detects failure. | When adding a regression check or establishing a new critical check. |
| `verify-compat <baseline> <candidate>` | An existing supported consumer can still use the public API, protocol, or schema after the change. | PRs touching public interfaces and before release. |
| `verify-privacy <target>` | After opt-out, a test server sees no disallowed telemetry. Also exercise the enabled path so the check can detect whether telemetry is working at all. | Changes to telemetry, consent, or the transport they use. |
| `verify-downstream <artifact>` | Install the candidate in a representative dependent project and run its normal workflow. | Changes to packages, plugins, SDKs, or other artifacts consumed downstream. |

For bug fixes, record the original reproduction, the log or experiment that locates the first incorrect behavior, and the passing rerun. A reviewer can challenge weak assertions or missing causal evidence. Keep automated review comments advisory.

## Platform and release checks

Run the same core scenario and assert the same outcome on each supported target. Keep one short feature map entry per scenario with the user entry point, command, expected result, and evidence.

| Target | Test environment |
| --- | --- |
| iOS | Build and install the simulator-compatible app on an iOS simulator. Verify signed distribution builds on a compatible device or distribution test channel when applicable. |
| Android | Build and install the app on an Android emulator. For an app bundle, test the installable APKs produced from the candidate bundle. |
| macOS | Install and launch the candidate package on macOS. Test each supported architecture where behavior differs. |
| Linux | Install and launch the candidate package in a clean environment for each supported package format or distribution combination. |
| Windows | Install and launch the candidate package on Windows. Test each supported architecture where behavior differs. |

| Suggested command | What passes | When to run |
| --- | --- | --- |
| `verify-package <target> <artifact> <scenario>` | Install the supplied candidate into a clean environment, launch it, and run the core scenario. Record the artifact version or digest with the result. | Every distributable release candidate. |
| `verify-upgrade <target> <previous-artifact> <candidate-artifact>` | Create representative data using the previous supported release, install the candidate over it, and confirm the data and core workflow survive. | From the second release onward. Include older starting versions when supported. |
| `verify-size <baseline-artifact> <candidate-artifact>` | Report the size delta between equivalent artifacts for the same target and architecture. Once a budget is set, fail on growth beyond it. | When mobile or desktop packages are built, with release checks once a budget exists. |

## Scheduled and risk-specific checks

| Suggested command | What passes | When to run |
| --- | --- | --- |
| `verify-repeat <target> <scenario> <iterations>` | Repeat the smoke scenario from clean state. Preserve first-attempt failures and report which iterations failed. | Scheduled or when diagnosing flaky workflows. |
| `verify-live <integration>` | A small operation against the real service succeeds with test credentials. Record dependency failures separately from product assertion failures. | Scheduled or before release for live integrations. |
| `verify-performance <baseline> <candidate>` | Comparable runs report startup, latency, or another user-relevant metric. Fail only against an agreed budget. | Performance-sensitive changes or scheduled comparisons. |
| `verify-fuzz <surface> <time-budget>` | Generated inputs or action sequences preserve documented properties. Record a seed and reproducer for failures. | Input-heavy or stateful surfaces, generally on a schedule. |
| `verify-invariant <model>` | A model checker finds no counterexample within the stated assumptions and bounds. A deliberate violation produces a counterexample. | Changes affecting a modeled invariant, such as exclusive retry ownership. |
| `verify-mutations <suite>` | Report deliberately changed behavior that the suite fails to detect. Review survivors for meaningful coverage gaps. | When assessing test strength in a critical area. |
| `verify-agent <scenario-set> <trials>` | Repeated runs meet the chosen outcome threshold. Assert state directly where possible and retain traces for failures. | Changes to model, prompts, or tools in products with variable agent behavior. |

Use nonzero exit status for a failed check. Report setup failures and skipped or inconclusive runs distinctly. Capture the command, target, revision, assertions, and useful logs or screenshots. Tear down test processes and state after each run.

## Public examples

- [VS Code repeated smoke tests](https://github.com/microsoft/vscode/blob/main/build/azure-pipelines/product-smoke-flaky.yml)
- [Firefox installer checks](https://github.com/mozilla-firefox/firefox/blob/main/taskcluster/kinds/linux-install-test/kind.yml) and [telemetry tests](https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/internals/integration_tests/index.html)
- [Chromium Android size checks](https://chromium.googlesource.com/chromium/src/%2B/main/docs/speed/binary_size/android_binary_size_trybot.md) and [stable interface presubmit](https://github.com/chromium/chromium/blob/main/PRESUBMIT.py#L3453)
- [React Native device and native runtime tests](https://github.com/react/react-native/blob/main/.github/workflows/test-all.yml)
- [Node.js live internet checks](https://github.com/nodejs/node/blob/main/.github/workflows/test-internet.yml)
- [Homebrew package and dependent checks](https://github.com/Homebrew/brew/blob/main/docs/Manpage.md#test-bot-options-formula)
- [PostgreSQL cross-version upgrade checks](https://www.postgresql.org/message-id/CAD5tBc%2BGrGjZydKHw1s-fdPOD%2BySw7RyjmQqMhwMVXgpK5pAnA%40mail.gmail.com)

The workflow and evidence conventions draw on [Lauren Tan's verification guide](https://github.com/cursor/plugins/blob/main/pstack/docs/guide/06-verify-and-ship.md) and [ai-engineering-skills' CI and diagnosis notes](https://github.com/j-ctang/ai-engineering-skills/blob/2cfc15724cd57a92fc38c12e2005c13fecab0918/docs/notes/phase-1-lessons.md#L49-L73).
