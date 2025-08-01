# About the project

## History

In 2023, I (Alexey Chekulaev) started to work on a project that consists of multiple microservices hosting in AWS.
First, I did not find a good GH action to upload files in S3, so I did one myself.
Then I felt a lack of GH action to publish Maven packages in AWS CodeArtifact, so I had to develop two more actions:
one to publish and one to resolve existing packages.
In spring 2025 I started my second project, and the number of services grew as a volume of similar code in release pipelines.
Then I combined all of them into a single action `agilecustoms/gha-release`.
But then (summer 2025) I decided to make it public and extracted stuff not specific to AgileCustoms into a separate action `agilecustoms/release`

## Current state

Currently (July 2025) this action is being used in two private AgileCustoms projects with over 20+ repositories,
which allows me (author) to have good coverage on different release scenarios

Did I think about a plugin system? Yes, I did. Having every type of artifact as a plugin would be great.
The problem is that right now this GH action is sort of a Frankenstein monster:
it is a crazy mix of other GH actions, my custom GH actions (composite and Node.js ones) and some shell scripts.
Ideally (maybe in the future) rewrite the whole thing in TypeScript or Go and then allow plugins with clear programmatic API.

So far idea is to add new types of artifacts as new steps in `action.yml` file, even though it is a "monolithic" approach

## Philosophy

1. Everything works out of the box, minimal setup is needed; inputs should have meaningful defaults
2. Implement features only when they are needed and thus can be tested!
3. Very thoughtful and detailed documentation, including examples for each use case
4. Verify inputs to detect errors as early as possible
5. Clear errors with suggestions on how to fix them

## why not just use `semantic-release`?

1. `semantic-release` in its core part has focus on release of npm packages, see `release_channel` which is npm specific
2. `semantic-release` as of June 2025 has no plugins to 1) upload files in S3, 2) publish Docker images to ECR, 3) publish maven in CodeArtifact
3. `semantic-release` is a library. To use it as a GH action, you need a wrapper, like [semantic-release-action](https://github.com/cycjimmy/semantic-release-action)
4. conceptually I found `semantic-release` somewhat hard to learn and configure,
I wanted to have a GH action that works fine by default and clear usecase-based documentation with examples
how to configure the action per specific use case

Internally `semantic-release` uses `conventional-changelog`. At some point I thought about using `conventional-changelog` directly,
but then I realized that `conventional-changlog` itself is a "Lego" - you need at least next 4 libraries:
conventional-changelog-angular, conventional-commits-parser, conventional-changelog-filter, conventional-changelog-writer.
So `semantic-release` does some heavy lifting to put them together.
