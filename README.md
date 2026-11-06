# DDE release workflow review 20261106-073925

Isolated reproduction of three review findings in the peeweep-test organization.
Source: current local changes in dde and .github@verify-group-version.

The release, recovery, version comparison and commit-status logic are unchanged.
Fixture wiring redirects member repositories and the guard's baseline repository
to these test repositories. Scheduled execution is disabled.

G2 contains two disposable Go fixtures. One member intentionally fails before its
release job while REVIEW_FAIL_BEFORE_RELEASE is true. G1 contains one DTK fixture.
No production repositories or existing peeweep-test release records are modified.

Source content hashes are recorded in source-sha256.json.
