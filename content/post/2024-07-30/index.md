---
title: "How to Modify an NPM Package When It Doesn't Meet Your Needs"
date: 2024-07-30T16:02:05+08:00
draft: false
author: ""
image: "https://cdn-images-1.medium.com/max/1600/1*ZS3iplWNdqvLSRzbbh8MzA.png"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

During the development process, if you find that the functionality of an NPM package does not fully meet the requirements, or there are bugs, how to deal with it?

There are scenarios where you might find a bug in an NPM package during development. Here's how you can handle such situations:

1. Submit an Issue or Pull Request: The first step is to submit an issue to the original author or fork the repository, make the necessary changes, and create a pull request. However, this approach has a significant downside - it can be time-consuming. If the author is strict or inactive, it might take a long time for your changes to be merged, which your project might not be able to afford.

2. Temporary or Alternative Solutions: In cases where modifying the source package is not feasible, you need a temporary or alternative solution. Here are two common scenarios:

- **Small Code Changes**: If the required modifications are minimal, consider using a patch.

- **Large Code Changes or Obfuscated Code**: If the changes are extensive or the package code is minified/obfuscated, making direct modifications might not be practical. In this case, you can modify the source code, change the package name, and republish it. Then, update your project to use this new package.

Directly modifying the code in node_modules is possible but not recommended, as these changes will be lost the next time you run npm install or update the package. Below are some recommended methods to handle this situation effectively:

## Using Fork

The most common method is to fork the source code. You can fork the repository of the third-party package on GitHub or other hosting platforms. After making modifications to the source code, publish the modified package to npm. If you do not want it to be public, you can set up a private npm package. Switch the package in your project to the one you published.

## Submitting PR

If you believe your modification could benefit other users, you can submit a pull request (PR) to the original package's maintainer. If the PR is accepted and merged, you can directly use the future versions of the official package without maintaining a fork.

## External Code Modification

This method involves not directly modifying the source code in `node_modules`, but using JavaScript features to modify the internal properties of the package during execution.

In simple terms, it involves using `defineProperty`, `prototype`, and other features to modify classes within the package. For example, in Vue 2.0, `defineProperty` is used to intercept and proxy data on component instances. In Vue projects, we often mount global properties and methods on the Vue root instance  `Vue.prototype.xxx = xxxx` in `main.js`.

## pnpm patch

`pnpm`'s patch command is inspired by a similar command in yarn.

First, execute `pnpm patch <pkg name>@<version>`. This command extracts the specified package to a temporary directory where you can freely edit it.
After making modifications, run `pnpm patch-commit <path> `(the path is the temporary directory created earlier, which can be long and hard to remember, but don't worry, the CLI will provide complete prompts) to generate a patch file and register it in your project using the `patchedDependencies` field.

## Aliases

pnpm provides an aliasing capability.

Suppose you published a new package called awesome-lodash and use lodash as an alias to install it:

```
pnpm add lodash@npm:awesome-lodash
```

Let's assume you use lodash all over your project. There is a bug in lodash that breaks your project. You have a fix but lodash won't merge it. Normally you would either install lodash from your fork directly (as a git-hosted dependency) or publish it with a different name. If you use the second solution you have to replace all the requires in your project with the new dependency name (`require('lodash') => require('awesome-lodash')`). With aliases, you have a third option.

Publish a new package called `awesome-lodash` and install it using lodash as its alias:

```
pnpm add lodash@npm:awesome-lodash
```

No changes in code are needed. All the requires of lodash will now resolve to awesome-lodash.

Sometimes you'll want to use two different versions of a package in your project. Easy:

```
pnpm add lodash1@npm:lodash@1
pnpm add lodash2@npm:lodash@2
```

Now you can require the first version of lodash via require('lodash1') and the second via require('lodash2').

This gets even more powerful when combined with hooks. Maybe you want to replace lodash with awesome-lodash in all the packages in node_modules. You can easily achieve that with the following `.pnpmfile.cjs`:

```
function readPackage(pkg) {
  if (pkg.dependencies && pkg.dependencies.lodash) {
    pkg.dependencies.lodash = 'npm:awesome-lodash@^1.0.0'
  }
  return pkg
}

module.exports = {
  hooks: {
    readPackage
  }
}
```

## patch-package

[patch-package](https://www.npmjs.com/package/patch-package#why-use-postinstall-postinstall-with-yarn)is a tool for fixing third-party dependencies, and it is very easy to use.

**Installation**：

```
$ npm i patch-package
# or with yarn
$ yarn add patch-package postinstall-postinstall
```

> **Note**: Most times when you do a yarn, yarn add, yarn remove, or yarn install (which is the same as just yarn) Yarn will completely replace the contents of your node_modules with freshly unpackaged modules. patch-package uses the postinstall hook to modify these fresh modules, so that they behave well according to your will.
>
> Yarn only runs the postinstall hook after yarn and yarn add, but not after yarn remove. The postinstall-postinstall package is used to make sure your postinstall hook gets executed even after a yarn remove.

**Usage:**

First make changes to the files of a particular package in your node_modules folder, then run

```
npx patch-package package-name
# or with yarn
yarn patch-package package-name
```

This will create a `patches` file in the root directory of your project that contains all the changes made to the files of the specified package in node_modules.  Inside will be a file called package-name+0.44.0.patch or something, which is a diff between normal old package-name and your fixed version. You can then commit this `patches` file along with your other changes. When you install dependencies again, patch-package will apply these patches automatically.

After completing the above steps, add the following to the `scripts` section in `package.json`:

```
"scripts": {
  "postinstall": "patch-package"
}
```

This way, when other team members pull the code and run `npm install` or `yarn install`, the patches will be automatically applied to the dependencies.

In simple terms, this solution records the code and location of the patch and uses npm's hook (the `postinstall` hook is triggered after `npm install`) to trigger the corresponding script after installing dependencies, overwriting the patches to the corresponding package in `node_modules`.

Of course, patches are version-specific, so you need to lock the version number. The downside is that if you need to upgrade, you have to repeat the process, but unless there are bugs or performance issues, you usually do not need to chase new versions.

## Advantages of patch-package

Although the above methods can solve specific scenarios through some tricks, they cannot avoid the problems caused by version upgrades. If the npm package is upgraded, it may cause errors in the previous modifications. Therefore, if you want to use the above methods, it is best to lock the version number. However, patch-package has the following features:

- **Version Mismatch**

  If the version of the package you installed does not match the version recorded in your previous patch, npx patch-package will directly report an ERROR: Failed to apply patch for package xxxx at path. The prompt helps you locate the problem more easily.

- **Space Saving**

  Using git diff to record patches is more space-saving than rewriting a source code, making it both safe and convenient.
