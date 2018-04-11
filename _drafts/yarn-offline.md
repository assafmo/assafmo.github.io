---
layout: post
title: Working offline with Yarn
date: 2018-04-11
---

NPM is horrible installing packages in an offline environment. Multiple times I found myself `npm install`ing on an internet machine just to copy `node_modules` to the offline environment and commit it with the entire project.

For a while [npmbox][npmbox-link] worked for me, but issues like [trying to reach the internet][unnpmbox-issue] would randomly appear and cripple my workflow.

I also tried [Sinopia][sinopia-docker-link], but couldn't consistently publish new or updated packages to it.

### Enter Yarn

With Yarn I'm able to consistently install packages in an offline environment. Their [original blogpost][yarn-original-blogpost] is helpful, but I encountered some edge case it doesn't cover. So this is my process for using Yarn in an offline environment.

### Configuring `yarn-offline-mirror`

#### On the internet machine:

```bash
yarn config set yarn-offline-mirror ~/yarn-offline-mirror/
```

#### On the offline machine:

```bash
yarn config set yarn-offline-mirror ~/yarn-offline-mirror/
```

Note: On the offline machine `~/yarn-offline-mirror/` can also be a shared folder or a git repository.

### Creating a new project

#### On the internet machine:

```bash
mkdir new-project/
cd new-project/
yarn add dep1@x.y.z [dep2...]
```

Then copy `new-project/yarn.lock`, `new-project/package.json` and `~/yarn-offline-mirror/` to the offline machine.

(`rm -rf new-project/` is ok now.)

#### On the offline machine:

```bash
mkdir new-project/
cp /path/to/imported/{yarn.lock,package.json} new-project/
cp -n /path/to/imported/yarn-offline-mirror/* ~/yarn-offline-mirror/
cd new-project/
yarn --offline
```

### Adding packages to an existing project

#### On the internet machine:

```bash
mkdir new-packages/
cd new-packages/
yarn add dep1@x.y.z [dep2...]
```

Then copy `new-packages/yarn.lock`, `new-packages/package.json` and `~/yarn-offline-mirror/` to the offline machine.

#### On the offline machine:

1.  Append the imported `yarn.lock` to the existing `yarn.lock`. I found that without this step Yarn sometimes fails to find the new packages in the offline cache:

    ```bash
    cat /path/to/imported/yarn.lock >> existing-project/yarn.lock
    ```

2.  Update `package.json` with the new dependencies. This means merging both `dependencies` fields together. An ugly one-liner I tend to use:

    ```bash
    cat existing-project/package.json <(cat existing-project/package.json /path/to/imported/package.json | jq '.dependencies' | jq -s 'add | {dependencies: .}') | jq -s add | sponge existing-project/package.json
    ```

3.  Update `yarn-offline-mirror`:

    ```bash
    cp -n /path/to/imported/yarn-offline-mirror/* ~/yarn-offline-mirror/
    ```

4.  Install the new packages. This step also fixes `existing-project/yarn.lock`.

    ```bash
    cd existing-project/
    yarn --offline
    ```

I found the if I skip steps 1 and 2, and in step 4 I do `yarn add --offline <dep1> [<dep2>...]` then Yarn might not find the new packages in the cache and fail. This bug still exists in version 1.5.1. I believe it is related to these GitHub issues: [[1]][yarn-offline-issue-1][[2]][yarn-offline-issue-2][[3]][yarn-offline-issue-3][[4]][yarn-offline-issue-4][[5]][yarn-offline-issue-5][[6]][yarn-offline-issue-6]

### Installing global packages

Yarn [discourages using global packages][yarn-no-global], so it's hard by design to install them.

1.  Find out where is the global installation location [[7]][yarn-global-location]:

    ```bash
    yarn global bin
    ```

    (Or set it with `yarn config set prefix <filepath>`)

2.  Add it to your path. E.g.:

    ```bash
    echo 'export PATH=${PATH}:'"$(yarn global bin)" >> ~/.bashrc
    source ~/.bashrc # reload
    ```

3.  Similar to [Creating a new project](#creating-a-new-project), but with a few subtle differences:

    #### On the internet machine:

    ```bash
    mkdir new-cli/
    cd new-cli/
    yarn add cli1@x.y.z [cli2...]
    ```

    Then copy `new-cli/yarn.lock` and `~/yarn-offline-mirror/` to the offline machine.

    (`rm -rf new-cli/` is ok now.)

    #### On the offline machine:

    ```bash
    cp /path/to/imported/yarn.lock .
    cp -n /path/to/imported/yarn-offline-mirror/* ~/yarn-offline-mirror/
    yarn global add --offline cli1@x.y.z [cli2...]
    rm -f ./yarn.lock
    ```

    Note: In this context we don't care about `packge.json`. We only need to make sure that Yarn can find `yarn.lock` in the current directory and that `~/yarn-offline-mirror/` has the required dependencies.

[npmbox-link]: https://github.com/arei/npmbox
[unnpmbox-issue]: https://github.com/arei/npmbox/issues/61
[yarn-original-blogpost]: https://yarnpkg.com/blog/2016/11/24/offline-mirror/
[yarn-offline-issue-1]: https://github.com/yarnpkg/yarn/issues/5454
[yarn-offline-issue-2]: https://github.com/yarnpkg/yarn/issues/5339
[yarn-offline-issue-3]: https://github.com/yarnpkg/yarn/issues/731
[yarn-offline-issue-4]: https://github.com/yarnpkg/yarn/issues/4909
[yarn-offline-issue-5]: https://github.com/yarnpkg/yarn/issues/4266
[yarn-offline-issue-6]: https://github.com/yarnpkg/yarn/issues/4899
[yarn-no-global]: https://stackoverflow.com/a/43901681
[yarn-global-location]: https://yarnpkg.com/lang/en/docs/cli/global/#defining-install-location
[sinopia-docker-link]: https://hub.docker.com/r/keyvanfatehi/sinopia/
