# Releases

## Overview

This document provides guidance to publish a new release for `ltestlib`.

## Steps before pushing a new tag

1. Create a feature branch for the changes;

2. Bump the version on `version` variable defined in the file `ltestlib.lua`;

> [!IMPORTANT]
>
> The version assigned to `version` must match the upcoming tag and also the following regex pattern: `^[0-9]+(\.[0-9]+)+$`. Thus, `0.2` is allowed, `0.2.0` is also allowed, but a single number `1` is **NOT** allowed (*unless this pattern is fixed on every spot at [./.github/workflows/publish.yml](./.github/workflows/publish.yml)*).

3. Bump the macro `PACKAGE_VERSION` on [Makefile.unix](./Makefile.unix) and [Makefile.win](./Makefile.win) to the exact **SAME VERSION** on `version` variable defined in the file `ltestlib.lua`;

4. On a Unix system, generate the rockspec for the new version: `make -f Makefile.unix create-rockspecs`;

5. Make sure the created rockspec (**CHANGE** `rockspecs/ltestlib-0.1.1-1.rockspec` below to the new name) prints the expected output running the following script from the terminal or command line:

    ```bash
    lua -e "dofile(arg[0]); print(); print('package  :', package); print('version  :', version); print('branch   :', source.branch); print('tag      :', source.tag); print('url      :', source.url); print('homepage :', description.homepage); print();" -- rockspecs/ltestlib-0.1.1-1.rockspec
    ```

6. Make sure `luarocks make` is able to build the project correctly for the fresh rockspec;

7. Create entries in the changelog describing the changes for the new version:

    * in the file [CHANGELOG.md](./CHANGELOG.md);
    * in the top of the file `ltestlib.lua`;

8. Commit the changes and push them to the remote repository;

## Tagging a new release

After all the previous steps were performed:

1. Create a new annotated tag (e.g.: `git tag -a "v0.1.1" -m "Release v0.1.1"`) changing `v0.1.1` in the previous command to contain the exact **SAME VERSION** on `version` variable defined in the file `ltestlib.lua`;

2. Push the new tag to the remote repository (e.g.: `git push origin v0.1.1`);

## Publish to LuaRocks and GitHub Releases

Once a new tag was released, select the `Publish` action on GitHub Web to trigger it manually:

1. In the UI, select the latest tag pushed in the previous step;
2. Hit the button to run the `Publish` action using the chosen tag.

> [!TIP]
>
> In order to account for intermittent failures (e.g.: connection issues, unavailable services) during the process to publish `ltestlib` to either LuaRocks or GitHub Releases, two input fields were added to skip the upload to each service in separate.
>
>   * `skip-luarocks-upload` (default: `false`): on GitHub Web UI, when the checkbox is ticked (this value is set to `true`), the `publish.yml` workflow does **NOT** upload to LuaRocks.org website.
>
>   * `skip-github-release` (default: `false`): on GitHub Web UI, when the checkbox is ticked (this value is set to `true`), the `publish.yml` workflow does **NOT** publish to GitHub Releases
