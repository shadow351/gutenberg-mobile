# WordPress Android Integration Guide

### Summary

WPAndroid by default integrates `react-native-bridge` as a binary dependency that's fetched from a remote maven repo.
The CI in this repo will deploy a new version each time a commit is pushed to `develop` or an open PR, as well as each time a tag is created.
This is the only required dependency.
Whenever a new version is deployed, we can simply update `ext.gutenbergMobileVersion` in [build.gradle](https://github.com/wordpress-mobile/WordPress-Android/blob/develop/build.gradle).

During development, we can use composite builds and dependency substitution to point WPAndroid to our local checkout of `gutenberg-mobile`.
To do this, we need to copy the [local-builds.gradle-example file](https://github.com/wordpress-mobile/WordPress-Android/blob/develop/local-builds.gradle-example) in WPAndroid renaming it to `local-builds.gradle` and change the `localGutenbergMobilePath` value.
Building the project as such will make WPAndroid build the source code for `react-native-bridge`, `react-native-aztec` and other sublibraries from the locally checked out folder.
It'll also expect the metro server to be running to fetch the JS bundle.

- [Work with local gutenberg-mobile checkout](#work-with-local-gutenberg-mobile-checkout)
- [Deploy a new version of `react-native-bridge` from CI](#deploy-a-new-version-of-react-native-bridge-from-ci)
- [Manually deploy a new version of `react-native-bridge`](#manually-deploy-a-new-version-of-react-native-bridge)
- [Update WPAndroid's binary version](#update-wpandroids-binary-version)
- [How to](#how-to)
    - [How to test a gutenberg-mobile PR in WPAndroid](how-to-test-a-gutenberg-mobile-pr-in-wpandroid)
    - [How to share WIP WPAndroid APK](how-to-share-wip-wpandroid-apk)
    - [How to checkout gutenberg-mobile to deployed binary version](#how-to-checkout-gutenberg-mobile-to-deployed-binary-version)

---

### Work with local gutenberg-mobile checkout

From your gutenberg-mobile checkout:

- Make sure you run `npm install` or `npm ci`
- Run metro server with `npm run start:reset`

From your WordPress-Android checkout:

- Copy [local-builds.gradle-example](https://github.com/wordpress-mobile/WordPress-Android/blob/develop/local-builds.gradle-example) renaming it to `local-builds.gradle`
- Update `localGutenbergMobilePath`to your local `gutenberg-mobile`checkout
- Run the project

#### How it works:

When [settings.gradle](https://github.com/wordpress-mobile/WordPress-Android/blob/develop/settings.gradle) finds `localGutenbergMobilePath` in `local-builds.gradle`, it'll substitute the below binary dependency with the local folder:

```
implementation "$rootProject.gradle.ext.gutenbergMobileBinaryPath:$rootProject.ext.gutenbergMobileVersion"
```

`react-native-bridge` is setup to include all the other projects, so no other action is necessary.

---

### Deploy a new version of `react-native-bridge` from CI

There are a few different ways to do this:

1. Open a new PR or push a new commit to an open PR which will be published as `<PR number>-<commit full SHA1>`
2. Merge a PR to `develop` which will be published as `<develop>-<commit full SHA1>`
3. Create a new tag which will be published as `<tag name>`

#### How it works:

CI will run the following commands:

- `npm ci`
- `npm run bundle:android`
- `gutenberg/packages/react-native-bridge/android/publish-aztec-and-bridge.sh <version>`

---

### Manually deploy a new version of `react-native-bridge`

_**IMPORTANT:** This should only be used for testing.
We shouldn't commit a manually deployed version to WPAndroid's `develop` because the process bypasses the guarantees CI would otherwise give us._

- Make sure you run `npm install` or `npm ci` first
- Run `npm run bundle:android` so the JS bundle is created
- Run `publish-aztec-and-bridge.sh` script in `react-native-bridge`, with the version name as its argument. i.e `./gutenberg/packages/react-native-bridge/android/publish-aztec-and-bridge.sh locally-built-by-<your_name>-<commit full SHA1>`

#### How it works:

Besides checking a few things, `publish-aztec-and-bridge.sh` script will clean the project, copy the bundled JS file into its assets, publish `react-native-aztec` and use that version to build and publish `react-native-bridge`.

---

### Update WPAndroid's binary version

- Find [the version deployed from CI](#deploy-a-new-version-of-react-native-bridge-from-ci) or [the version you've manually deployed](#manually-deploy-a-new-version-of-react-native-bridge)
- Update `ext.gutenbergMobileVersion` property in [build.gradle](https://github.com/wordpress-mobile/WordPress-Android/blob/develop/build.gradle) to the new version

In order to test this, make sure `localGutenbergMobilePath` in your `local-builds.gradle` file is commented out as otherwise the binary version will be ignored.

---

## How to

### How to test a gutenberg-mobile PR in WPAndroid

Assuming that there is no open WPAndroid PR yet:

1. Setup WPAndroid to [work with local gutenberg-mobile checkout](#work-with-local-gutenberg-mobile-checkout)
2. Switch `gutenberg-mobile` to the branch you're testing
3. Run the app

---

### How to share a WIP WPAndroid APK

1. Either (a) open a `gutenberg-mobile` PR which will [publish a new version](#deploy-a-new-version-of-react-native-bridge-from-ci) or (b) [manually publish a new version](#manually-deploy-a-new-version-of-react-native-bridge)
2. Open a WPAndroid PR [updating the binary version](#update-wpandroids-binary-version)
3. Peril will publish a comment to the WPAndroid PR with a link to the APK

Alternatively, once the `react-native-bridge` version is published, we can locally create the APK.
**Note that there is no way to use composite build to create a WPAndroid APK that can be shared. This is because composite build only works with the local metro server.**

---

### How to checkout gutenberg-mobile to deployed binary version

Whenever the CI publishes `react-native-bridge`, it will either include the full commit hash or the tag in its name.
So, we can checkout simply checkout the commit hash or the tag.
