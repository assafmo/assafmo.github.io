---
layout: post
title: Working offline with Yarn
date: 2018-04-10
---

NPM is horrible installing packages in an offline environment. Multiple times I found myself `npm install`ing on an internet machine just to copy `node_modules` to the offline environment and commit it with the entire project.

For a while [npmbox](npmbox-link) worked for me, but issues like [trying to reach the internet](unnpmbox-issue) would randomly appear and cripple my workflow.

### Enter Yarn

With Yarn I'm able to consistently install packages in an offline environment. Their [original blogpost](yarn-original-blogpost) is helpful, but I encountered some edge case it doesn't cover. So this is my process for using Yarn in an offline environment.

### Set yarn-offline-mirror

#### On the internet machine:

```bash
yarn config set yarn-offline-mirror ~/yarn-offline-mirror/
```

#### On my offline machine:

```bash
yarn config set yarn-offline-mirror ~/yarn-offline-mirror/
```

Note: On the offline machine, `~/yarn-offline-mirror/` can also be a shared folder or a git repository.

### Creating a new project

#### On the internet machine:

```bash
mkdir new-project/
cd new-project/
yarn add <dep1> [<dep2>...]
```

Then copy `new-project/yarn.lock`, `new-project/package.json` and `~/yarn-offline-mirror/` to the offline machine.

#### On the offline machine:

```bash
mkdir new-project/
cp /path/to/imported/{yarn.lock,package.json} new-project/
cp -n /path/to/imported/yarn-offline-mirror/* ~/yarn-offline-mirror/
cd new-project/
yarn --offline
```

### Adding a package to an existing project

### Installing global packages

[npmbox-link]: https://github.com/arei/npmbox
[unnpmbox-issue]: https://github.com/arei/npmbox/issues/61
[yarn-original-blogpost]: https://yarnpkg.com/blog/2016/11/24/offline-mirror/
