# Maintaining and releasing the plugin

## Main branch

Use a feature branch and pull request. Main requires up-to-date passing GitHub Actions tests,
resolved conversations, and blocks force pushes and deletion. Rules apply to administrators too.
No second human reviewer is required, so a sole maintainer can merge their own green pull request.
Keep required check names aligned with `.github/workflows/test.yml` when changing the matrix.

Do not bypass failed tests to publish. Dependency-update pull requests need their own compatibility
review; a major dependency bump is not routine release housekeeping.

## One-time npm trusted publisher setup

Sign in as an owner of `medusa-payment-peach-payments`. In the package's settings, configure a
GitHub Actions trusted publisher with these exact, case-sensitive values:

- Organization or user: `max-bissolati`
- Repository: `medusa-payment-peach-payments`
- Workflow filename: `release.yml`
- Environment: leave empty; the workflow does not declare a GitHub environment.

The workflow uses GitHub-hosted runners, Node 24, npm 11.18.0 and `id-token: write`. It does not need
an `NPM_TOKEN` secret. npm trust must be configured on npm itself; GitHub permissions do not create
that relationship. Do not claim this setup is verified until a real publish succeeds.

See [npm trusted publishing](https://docs.npmjs.com/trusted-publishers/). If publication reports
`ENEEDAUTH`, check the npm trust fields and Node/npm versions before retrying. Do not paste tokens
into issues, logs or committed configuration.

## Release checklist

1. Confirm the current npm version and GitHub release/tag state. Do not republish or move an
   existing version tag. v0.1.5 was already on npm even though its GitHub publish run failed.
2. Update `package.json`, `package-lock.json`, the changelog and relevant documentation for a new
   version. Preserve the README banner and badges. Review the actual diff.
3. Run installation, typecheck, tests, build and the artifact checks from the workflows. Inspect
   the packed contents: built provider JavaScript and declarations must exist, with no source,
   examples or spec files. Local success alone does not establish hosted publishing credentials.
4. Push the branch and merge its pull request only after required CI passes. Confirm the merged
   commit contains the intended package version.
5. With publication authorized, tag the intended main commit `v<package version>` and push that
   new tag. The release workflow validates the tag/version match and builds/tests before publishing
   with provenance. Wait for the actual workflow result.
6. Verify `npm view medusa-payment-peach-payments version dist-tags --json`, then publish the
   corresponding GitHub release with useful notes and the intended Latest designation. A Git tag,
   npm package and GitHub release are three separate states; verify each.

If npm publication succeeds but another step fails, confirm the registry before retrying anything.
Never unpublish or overwrite a version to make a historical workflow indicator green. Record an
unresolved account-side setup or release failure clearly in `BACKLOG.md`.
