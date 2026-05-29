# License Policy

## OSS license

Every public Phoxal repository is licensed under **AGPL-3.0-only**
(SPDX: `AGPL-3.0-only`). Each repo carries a root `LICENSE` file with the
full AGPLv3 text.

The AGPL applies to:

- every public `phoxal-*` crate published to crates.io;
- every Phoxal binary (CLI, simulator, operator tools);
- every Phoxal runtime image published to GHCR;
- every scaffold emitted by `phoxal-cli create`.

## Commercial license

Phoxal offers a commercial license for downstream products that cannot
meet the AGPL source-disclosure obligations. The OSS license remains
AGPLv3 — there's no relicensing of the public crates. The commercial
license is granted per-customer and bypasses the AGPL section 13 network-use
trigger for that customer's product.

For commercial licensing inquiries, see each repo's `COMMERCIAL.md` (where
present) or [phoxal.com](https://phoxal.com).

## Inbound contributions

All contributions are submitted under the same AGPL-3.0-only license as the
target repository. Contributors sign off commits per the **Developer
Certificate of Origin** (DCO) — see
[CONTRIBUTING.md](CONTRIBUTING.md).

DCO sign-off certifies that the contributor has the right to submit the
change under AGPL-3.0-only. It does **not** grant Phoxal the right to
relicense the contribution under non-AGPL terms.

The Phoxal team may move from DCO to a Contributor License Agreement (CLA)
before accepting external contributions at scale, in order to keep the
commercial dual-license path open for contributed code. The decision is
tracked under
[phoxal/robot-framework#817](https://github.com/phoxal/robot-framework/issues/817);
until that work concludes, please assume DCO and AGPL-3.0-only.

## Dependency policy

When adding a Cargo dependency or vendoring third-party code, check the
license:

| License | OK to depend on? |
|---|---|
| MIT, Apache-2.0, BSD-2/3, ISC, Unlicense, MPL-2.0 | Yes — standard permissive |
| AGPL-3.0, GPL-3.0 | Compatible inbound — OK to depend on (we're AGPL ourselves) |
| LGPL-2.1 / LGPL-3.0 | Usually OK; flag in PR for review |
| GPL-2.0-only (not "or later") | Flag for review — AGPLv3 + GPLv2-only requires care |
| Commercial / proprietary / source-available non-OSS | **No** without explicit team review |
| Unclear / unspecified license | **No** until clarified upstream |
| CC0 / public domain | Yes (treat as permissive) |
| CC-BY / CC-BY-SA (data, not code) | Case-by-case — only for data assets, never for executable code |

When in doubt, open an issue describing the dependency and the case for
adding it before merging.

## License headers

Source files do not need per-file license headers. The root `LICENSE` and
the `license` field in each `Cargo.toml` are authoritative. Adding SPDX
headers to source files is fine but not required.

## Stability

This policy may evolve. Material changes will be announced in the affected
repos' changelogs and on [phoxal.com](https://phoxal.com).
