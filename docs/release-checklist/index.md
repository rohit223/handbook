---
layout: doc
title: Release checklist
description: Tasks performed during the process of tagging a release.
category: Contributing
quick_links:
  - Major/minor releases
  - Patch releases
---

This page describes the list of activities that developers must perform to produce a new release of WP-CLI.

## Major/minor releases

### Updating WP-CLI

Make sure that the contents of [VERSION](https://github.com/wp-cli/wp-cli/blob/master/VERSION) are changed to latest.

### Locking php-cli-tools version

php-cli-tools is set to `dev-master` during the development cycle. During the WP-CLI release process, `composer.json` should be locked to a specific version. php-cli-tools may need a new version tagged as well.

### Updating the contributor list

Use `./utils/contrib-list` to see new contributors. Update the `.mailmap` file so that the names match their github handles.

When done, use `sort .mailmap -f -u -o .mailmap` to only add new contributors.

### Updating the Phar build

1) Create a git tag and push it.

2) Create a stable [Phar build](https://github.com/wp-cli/builds/tree/gh-pages/phar):

    cd wp-cli-builds/phar
    cp wp-cli-nightly.phar wp-cli.phar
    md5 -q wp-cli.phar > wp-cli.phar.md5
    shasum -a 512 wp-cli.phar | cut -d ' ' -f 1 > wp-cli.phar.sha512

3) Sign the release with GPG. See <https://github.com/wp-cli/wp-cli/issues/2121>

    gpg --output wp-cli.phar.gpg --sign wp-cli.phar

4) Create a release on Github: <https://github.com/wp-cli/wp-cli/releases>. Make sure to upload the Phar from the builds directory.

### Updating the Debian build

1) Run this script: <https://github.com/wp-cli/wp-cli/blob/master/utils/wp-cli-updatedeb.sh>

2) Symlink the latest deb: `ln -sfv php-wpcli_${VERSION}_all.deb php-wpcli_latest_all.deb`

3) Commit file to builds repo

### Updating the Homebrew formula

A pull request must be submitted to the Homebrew repo. See <https://github.com/Homebrew/homebrew-php/pull/1687#issuecomment-98408399> for background.

### Updating the website

See <https://github.com/wp-cli/wp-cli.github.com#readme>

### Writing the release post

Use `./utils/contrib-list -l` to generate the list of contributors.

## Patch releases

Creating a patch release (e.g. 0.23.x) is bit different of a process than creating a major or minor release. At a high-level, here are the steps involved:

1) Create a new release branch from the last tagged patch release:

    $ git checkout v0.23.0
    Note: checking out 'v0.23.0'
    You are in 'detached HEAD' state. You can look around, make experimental
    changes and commit them, and you can discard any commits you make in this
    state without impacting any branches by performing another checkout.
    $ git checkout -b release-0-23-1
    Switched to a new branch 'release-0-23-1'

2) Cherry-pick existing commits to the new release branch.

Because patch releases should just be used for bug fixes, you should first fix the bug on master, and then cherry-pick the fix to the release branch. It's up to your discretion as to whether you cherry-pick the commits directly to the release branch *or* create a feature branch and pull request against the release branch.

3) Update `VERSION` on the release branch to the new release version.

4) **PROCEED WITH EXTREME CAUTION**. While the normal release process yields a built, fully-tested Phar file, the patch release process does not because the build system only pushes the Phar file on the master branch. As such, you need to manually build the Phar file for distribution.

    php -dphar.readonly=0 utils/make-phar.php wp-cli.phar --quiet

When you do so, make sure you're using the appropriate Composer dependency versions for the release, not the master branch you normally work from. Once you've verified the built Phar, you'll need to copy it over to the builds repo.

5) Follow all of the other relevant release steps.