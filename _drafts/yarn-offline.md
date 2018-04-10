---
layout: post
title: Working offline with Yarn
date: 2018-04-10
---

NPM is horrible installing packages in an offline environment. Multiple times I found myself `npm install`ing in an internet machine just to copy `node_modules` to the offline environment and commit it with the entire project.

For a while npmbox [[1]][npmbox-link] worked for me, but issues like trying to reach the internet [[2]][unnpmbox-issue] would randomly appear and cripple my workflow.

### Enter Yarn

With Yarn I'm able to consistently install packages in an offline environment. This is my process.

### Set yarn-offline-mirror

### Creating a new project

### Adding a package to an existing project

### Installing global packages

[npmbox-link]: https://github.com/arei/npmbox
[unnpmbox-issue]: https://github.com/arei/npmbox/issues/61
