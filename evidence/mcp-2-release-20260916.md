# PHP core 2.1.1 release preparation

Prepared 2026-09-16 09:46 AEST. Status: coded and tested; not merged, tagged, or published. ROOT owns those remaining steps and public-index acceptance.

## Identity and bounded delta

- Split base: `8312a2436c499258642778bff6b0b87f2cd373be` (`origin/main`, freshly fetched).
- Package commit: `2c004dfbc816c5c2acb67fbea3001eedcdad9130`.
- Immutable source: `Sendmux/sendmux-sdk@fb6085ce643c117dc3f2beb29c4fc57c303272b7`, `packages/php/core/`.
- Package diff: `CHANGELOG.md`, `README.md`, `composer.json`; 14 insertions, 2 deletions, no package-file additions or removals. This evidence file is the only tracked addition.
- All 12 package files match the source, with one explicit exception: `CHANGELOG.md:3` replaces `Unreleased` with `2.1.1 (2026-09-16)`, following this split's existing release-heading style.
- All eight `src/` files and `LICENSE` are byte-identical to both the source and split base. No public method, auth surface, retry behavior, or runtime path changed.

Canonical source and release requirements: immutable monorepo `scripts/check-php-splits.mjs:8-87` and the coordinated PHP split preflight. This copies the already-approved package instead of reconstructing it. The manifest intentionally contains no `version`.

## Correctness trace

- `composer.json:8` carries the approved Guzzle floor `^7.15.5` from the monorepo.
- `README.md:18-23` gates the `^2.1.1` installation example on public Packagist availability.
- `README.md:28` preserves the complete custom `FileCookieJar` / `SessionCookieJar` migration warning: back up state, re-authenticate into a fresh jar, and do not guess `HostOnly`.
- `CHANGELOG.md:5-7` preserves the source Security entry and its upgrade-warning link. Earlier release history is unchanged.
- Prior source security/adoption review is recorded in the immutable monorepo `evidence/mcp-2-modernisation-20260915.md:64-77`; its older verification ranges remain older ranges, not fresh claims about this split.

## Candidate verification

An isolated Composer consumer mirrored this split using a path repository and an explicit fixture version `2.1.1`, as the existing split checker does. It installed normal public dependencies with `composer install --no-interaction --no-progress --prefer-dist`; dependency resolution and auditing remained enabled. This fixture version is not a published-version receipt.

| Check | Result |
| --- | --- |
| Split and installed package `composer validate --strict` | Both valid, exit 0; no package manifest warning. |
| Normal dependency resolution | Guzzle 7.15.5, PSR7 2.13.1, PHPUnit 11.5.56 installed on PHP 8.4.10. |
| `composer check-platform-reqs` | PHP and all eight listed extensions passed. |
| `composer audit --locked --format=json` | No advisories; no abandoned packages. |
| Existing `CoreTest` and `OAuthRetryTest` | 17 tests, 50 assertions, zero failures/errors/skips. |
| Source and installed provenance | All 12 installed package files equal the candidate bytes; all eight reflected core classes load from the isolated consumer's `vendor/sendmux/core/src`. |
| PHP lint | All eight runtime files passed. |
| `git diff --check` | Passed. |

The temporary consumer's own `composer validate --strict` exits 1 only for its deliberate exact `sendmux/core:2.1.1` fixture pin. The warning and exit code are retained. The publishable manifest retains its normal ranges and passes strict validation.

The unchanged PHPUnit files cover credential-surface acceptance/rejection, headers, cursor iteration and missing-cursor rejection, error mapping, retry eligibility, elapsed budgets, server delays, idempotency preservation, and explicit non-retryable responses. Test inputs and their helper classes match the immutable source byte-for-byte.

Tests: +0, -0; no new implementation or duplicate test was introduced. Existing behavior tests were reused for release packaging. The full monorepo suite and live APIs were not run by this delegation.

Journeys: not applicable to this metadata/doc package delta. The Composer consumer exercised the installed core interface offline; no browser or production request was needed.

## Evidence and resources

Raw receipts live in MAIN `.claude/artifacts/mcp-2-release/`:

- `source-parity.json`, `test-source-parity.json`, `package-delta.patch`: immutable source, candidate, installed-byte comparisons and exact patch.
- `consumer-install.log`, `consumer-composer.json`, `consumer-composer.lock`, `installed-{core,guzzle,psr7}.json`, `autoload-provenance.json`: resolved dependency and candidate identity.
- `manifest-validation.log`, `installed-manifest-validation.log`, `installed-manifest-validation.exit`, `consumer-validation.log`, `consumer-validation.exit`, `platform.log`, `composer-audit.json`, `phpunit-core.{log,xml}`, `php-lint.log`: verification outputs.
- `remote-refs-before.txt`, `github-main-before.json`, `packagist-before.json`: live base and target-version absence receipts.
- `source-tree.tar.gz`, `consumer-inputs.tar.gz`: retained source and test inputs for reproducibility; dependencies can be reconstructed from the lock.
- `cleanup.json`, `reaper-dry-run.txt`, `main-*-after.txt`, `auth-surface-*-after.txt`: owned-resource cleanup and preservation of unrelated work.

Removed and verified absent: exact task-owned directories `source.1Oiqh2` and `consumer.oWLsrU` under that artifact root. Composer process session `17644` completed; no persistent process, browser, server, container, or network tunnel was created. Their non-regenerable inputs remain in the archives above.

The release worktree remains for its PR. MAIN retains its original head `ff4c6a757ad1f058ee7bd412ccb8009876a99cbd` and user-owned untracked `.claude/` / `.gitignore`. The unrelated auth-surface worktree retains head `7a003a1a6d6b5b9475cac01b8260e0abe3591e97`; no files there were changed. Reaper ran in report-only mode: no existing worktree or resource was removed.

## Outstanding release gates

At preparation, remote `main` and Packagist `v2.1.0` both identify the base above; remote tag `v2.1.1` and public Packagist version 2.1.1 are absent. ROOT must settle live PR reviews/checks, merge, tag the exact merge SHA, prove Packagist ingestion at that SHA, and run the native published consumer before declaring this package released.

Independent review: Ready for the bounded preparation/PR path; 0 Critical, 0 Important, 0 Minor findings. Reviewed base `8312a2436c499258642778bff6b0b87f2cd373be` through package head `2c004dfbc816c5c2acb67fbea3001eedcdad9130` and the evidence draft. The reviewer independently checked Git/archive hashes and inspected receipts; it did not rerun Composer/PHPUnit or prove publication. Report: `.claude/artifacts/mcp-2-release/independent-review.md`, SHA256 `769c2e9e344fe2a40f37d222086631d9e86622c8e34b0ec1f84d7c31cee4f4eb`.

Parked: none in this package delta. The pre-existing auth-surface work remains outside this task.
