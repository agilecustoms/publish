# Release types

There are four types of releases:
- **normal release** (they don't even have a special name): a regular release that is ready for production use,
typically made from `main`, `master`, or `release` branch (sometimes `develop`)
- **maintenance release**: same as normal release, but made from a maintenance branch
(such as `1.x.x` given `2.x.x` development is in `main`) to support users who did not upgrade to the latest version yet
- **prerelease**: here you develop a next (often major, sometimes minor) version, typically made from a branch `next`
- **dev-release**: a temporary release that developers do by their own from a feature branch

The difference between **prerelease** and **dev-release** is not obvious, but it is important to understand:

_prerelease_ is an industry standard, though some tools use different terms
like "prerelease" (GitHub), distribution tags (npm), suffix "-SNAPSHOT" (maven), suffixes "-beta", "rc" (Gradle, NuGet, PyPI, pip).
Idea: release a version widely available for testing and potential to become a next major release

_dev-release_ is just a way to overcome the inability to spin up the entire env locally.
So you have your feature branch and want to deploy it in sandbox or dev environment for dev testing or POC.

Imagine a team of developers working on a prerelease 2.0.0-alfa based off the branch `next`.
Each developer (rarely two) creates a feature branch. And then, if this feature requires testing in cloud,
the developer may decide to create a dev-release based off the branch name (`dev-feature1`), then test it and merge to `next` branch

The table below shows a comparison of different release types:

| Name                                             | normal release and maintenance release          | prerelease                                      | dev-release                                                |
|--------------------------------------------------|-------------------------------------------------|-------------------------------------------------|------------------------------------------------------------|
| intention                                        | use in production                               | beta testing                                    | dev testing                                                |
| best use for                                     | software packages and end applications          | software packages and end applications          | end applications                                           |
| adoption                                         | widely                                          | widely                                          | popular in enterprise                                      |
| versioning                                       | semantic-release used to determine next version | semantic-release used to determine next version | version = branch name, override on each push, no increment |
| auto deletion                                    | N/A                                             | ❌️                                              | ✅                                                          |
| number of developers                             | many                                            | many                                            | typically one                                              |
| release notes and changelog                      | ✅                                               | ✅                                               | ❌️                                                         |
| floating tags                                    | major, major.minor, latest                      | ?                                               | ❌️                                                         |
| semantic tags `fix:`, `feat:` in commit messages | at least one commit w/ such tag is required     | at least one commit w/ such tag is required     | not required                                               |

## Maintenance release

Consider a repo with current version `2.x.x` in `main` branch and `1.x.x` with legacy version support.
You need to place a file `.releaserc.json` in the repo root with the following content:
```json
{
  "branches": [
    "main",
    "1.x.x"
  ]
}
```
And also make sure your release workflow (e.g. `.github/workflows/release.yaml`) is triggered on `1.x.x` branch as well:
```yaml
on:
  push:
    branches:
      - main
      - 1.x.x
```

For **non-standard** maintenance branch name, like `support`, you need following `.releaserc.json`:
```json
{
  "branches": [
    "main",
    {
      "name": "support",
      "range": "1.x.x"
    }
  ]
}
```
Read about maintenance branches in [semantic-release documentation](https://semantic-release.gitbook.io/semantic-release/usage/workflow-configuration#maintenance-branches)

## Prerelease

Read about prerelease branches in [semantic-release documentation](https://semantic-release.gitbook.io/semantic-release/usage/workflow-configuration#pre-release-branches)

## Dev release

Dev release allows publishing artifacts temporarily for testing purposes:
you push your changes to the feature branch, the branch name becomes this dev-release version:
- semver is _not_ generated
- no git tags created — your branch name is all you need
- if branch name is `dev/feature` then the version will be `dev-feature`
- parameter `dev-branch-prefix` (default value is `dev/`) enforces branch naming for dev releases,
it helps to automatically dispose dev-release artifacts. Set to empty string to disable such enforcement
- for each artifact type, dev-release might have different semantics, see `dev-release` section for each artifact type

Example of 'dev-release' usage with AWS S3:
```yaml
steps:
  - name: Release
    uses: agilecustoms/release@v1
    with:
      aws-account: ${{ vars.AWS_ACCOUNT_DIST }}
      aws-region: us-east-1
      aws-role: 'ci/publisher' # default
      aws-s3-bucket: 'mycompany-dist'
      dev-release: true
      dev-release-prefix: 'dev/' # default
```

**dev-release motivation**

_How did we live w/o a dev-release before?
We do use microservices in our team, and we just have a CI/CI pipeline that can build a feature branch and deploy it to a dev server.
So we do not need dev-release, right?_

Build-and-deploy is kind of "shortcut" and it may work in simple scenarios.
The reality is that a system is a _combination_ of multiple services and true deployment means a deployment of a combination of services!
Imagine a system that consists of two services A and B, and there is a repo C storing the current combination: `A@1`, `B@2`.
And only C has a "Deploy" button! 
Now, you want to make a change in service A and test it, but since the only way to deploy a system is to deploy both services together,
you must create a temporary release of A. This is a dev-release!
