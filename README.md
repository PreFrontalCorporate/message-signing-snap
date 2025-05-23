# MetaMask Message Signing Snap

This Snap allows certain messages to be signed & verified through a randomly generated private key.

## Installation

`yarn add @metamask/message-signing-snap`

or

`npm install @metamask/message-signing-snap`

## Usage

See the [RPC API](./docs/RPC.md) for more information on how to interact with the snap.

### Example usa of the signing API

```ts
// Example: Get Signing Public Key
const publicKey: string = await ethereum.request({
  method: 'wallet_invokeSnap',
  params: {
    // or local:http://localhost:8080 if running snap locally
    snapId: 'npm:@metamask/message-signing-snap',
    request: {
      method: 'getPublicKey',
      params: {
        entropySourceId: '...', // optionally choose a specific SRP
      },
    },
  },
});

// Example: Sign a message
const signature: string = await ethereum.request({
  method: 'wallet_invokeSnap',
  params: {
    // or local:http://localhost:8080 if running snap locally
    snapId: 'npm:@metamask/message-signing-snap',
    request: {
      method: 'signMessage',
      params: {
        message: 'metamask:my message to sign',
        entropySourceId: '...', // optionally choose a specific SRP
      },
    },
  },
});
```

The `signature` can be verified against the `publicKey`.

### Example usage of the encryption capabilities

Alice, the user of this functionality, would call the
`getPublicEncryptionKey` method and obtain a hex encoded X25519 public
key, which they would broadcast to Bob.

```ts
const encryptionPublicKeyHex: string = await ethereum.request({
  method: 'wallet_invokeSnap',
  params: {
    snapId: 'npm:@metamask/message-signing-snap',
    request: {
      method: 'getEncryptionPublicKey',
      params: {
        entropySourceId: '...', // optionally choose a specific SRP
      },
    },
  },
});
```

Bob would then use a piece of code similar to:

```ts
import { encrypt } from '@metamask/eth-sig-util';

const encrypted = encrypt({
  encryptionPublicKeyHex,
  data: 'Hello Alice!',
  version: 'x25519-xsalsa20-poly1305',
});
```

... and obtain an object such as:

```json5
// encrypted data from Bob
{
  version: 'x25519-xsalsa20-poly1305',
  nonce: '8jFlr2cYdkyBIookw6akE8lA0f+odanN',
  ephemPublicKey: 'UOPWFRaPdY6kKDEoM/ovCBCT2p4PSx6MMdOgNzA2gC0=',
  ciphertext: '2UNvHmxnmOHtI9vhf7gfeD/J7Q/q6vqjEQqY',
}
```

Bob would send this object to Alice who can then call the
`decryptMessage` method of the snap to get the message.

```ts
const decryptedMessage: string = await window.ethereum.request({
  method: 'wallet_invokeSnap',
  params: {
    snapId: 'npm:@metamask/message-signing-snap',
    request: {
      method: 'decryptMessage',
      params: {
        data: {
          version: 'x25519-xsalsa20-poly1305',
          nonce: '8jFlr2cYdkyBIookw6akE8lA0f+odanN',
          ephemPublicKey: 'UOPWFRaPdY6kKDEoM/ovCBCT2p4PSx6MMdOgNzA2gC0=',
          ciphertext: '2UNvHmxnmOHtI9vhf7gfeD/J7Q/q6vqjEQqY',
        },
        entropySourceId: '...', // optionally choose a specific SRP
      },
    },
  },
});
```

Sometimes Alice may not be able to know which SRP she should use to decrypt the message. In that situation the snap will
try all available SRPs for the decryption.

## API

See our documentation:

- [RPC API](./docs/RPC.md) for more information on how to interact with the snap.

## Testing

See our documentation

- [Testing Docs](./docs/testing.md) on how to call this pre-installed snap.

## Contributing

### Setup

- Install the current LTS version of [Node.js](https://nodejs.org)
  - If you are using [nvm](https://github.com/creationix/nvm#installation) (recommended) running `nvm install` will
    install the latest version and running `nvm use` will automatically choose the right node version for you.
- Install [Yarn v3](https://yarnpkg.com/getting-started/install)
- Run `yarn install` to install dependencies and run any required post-install scripts

### Testing and Linting

Run `yarn test` to run the tests once. To run tests on file changes, run `yarn test:watch`.

Run `yarn lint` to run the linter, or run `yarn lint:fix` to run the linter and fix any automatically fixable issues.

### Release & Publishing

The project follows the same release process as the other libraries in the MetaMask organization. The GitHub Actions [
`action-create-release-pr`](https://github.com/MetaMask/action-create-release-pr) and [
`action-publish-release`](https://github.com/MetaMask/action-publish-release) are used to automate the release process;
see those repositories for more information about how they work.

1. Choose a release version.

- The release version should be chosen according to SemVer. Analyze the changes to see whether they include any
  breaking changes, new features, or deprecations, then choose the appropriate SemVer version.
  See [the SemVer specification](https://semver.org/) for more information.

2. If this release is backporting changes onto a previous release, then ensure there is a major version branch for that
   version (e.g. `1.x` for a `v1` backport release).

- The major version branch should be set to the most recent release with that major version. For example, when
  backporting a `v1.0.2` release, you'd want to ensure there was a `1.x` branch that was set to the `v1.0.1` tag.

3. Trigger the [
   `workflow_dispatch`](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#workflow_dispatch)
   event [manually](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow) for the
   `Create Release Pull Request` action to create the release PR.

- For a backport release, the base branch should be the major version branch that you ensured existed in step 2. For a
  normal release, the base branch should be the main branch for that repository (which should be the default value).
- This should trigger the [`action-create-release-pr`](https://github.com/MetaMask/action-create-release-pr) workflow
  to create the release PR.

4. Update the changelog to move each change entry into the appropriate change
   category ([See here](https://keepachangelog.com/en/1.0.0/#types) for the full list of change categories, and the
   correct ordering), and edit them to be more easily understood by users of the package.

- Generally any changes that don't affect consumers of the package (e.g. lockfile changes or development environment
  changes) are omitted. Exceptions may be made for changes that might be of interest despite not having an effect upon
  the published package (e.g. major test improvements, security improvements, improved documentation, etc.).
- Try to explain each change in terms that users of the package would understand (e.g. avoid referencing internal
  variables/concepts).
- Consolidate related changes into one change entry if it makes it easier to explain.
- Run `yarn auto-changelog validate --rc` to check that the changelog is correctly formatted.

5. Review and QA the release.

- If changes are made to the base branch, the release branch will need to be updated with these changes and review/QA
  will need to restart again. As such, it's probably best to avoid merging other PRs into the base branch while review
  is underway.

6. Squash & Merge the release.

- This should trigger the [`action-publish-release`](https://github.com/MetaMask/action-publish-release) workflow to
  tag the final release commit and publish the release on GitHub.

7. Publish the release on npm.

- Wait for the `publish-release` GitHub Action workflow to finish. This should trigger a second job (`publish-npm`),
  which will wait for a run approval by the [`npm publishers`](https://github.com/orgs/MetaMask/teams/npm-publishers)
  team.
- Approve the `publish-npm` job (or ask somebody on the npm publishers team to approve it for you).
- Once the `publish-npm` job has finished, check npm to verify that it has been published.
