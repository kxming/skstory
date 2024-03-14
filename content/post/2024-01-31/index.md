---
title: 'Knowledge Points Analysis on Development Environment'
date: 2024-01-31T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*OZL33xhjFl0XFUmv"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "The development environment of engineers determines their development efficiency...."
---

The development environment of engineers determines their development efficiency. Common development environment configurations are also one of the interview examination points.

# Knowledge Sorting

- IDE

- Git

- Basic Linux commands

- Front-end build tools

- Debugging methods

This section will focus on the basic usage of Git, code deployment, and commonly used Linux commands in development. Then, we will introduce front-end build tools with webpack as an example. Finally, we will introduce how to capture packets to solve online problems. These are all commonly used knowledge in daily development and interviews.

---

## IDE

> Topic: What IDE do you usually use for programming? What are the ways to improve efficiency?

The most commonly used IDEs for front-end development are Webstorm, Sublime, Atom, and VSCode. We can go to their official websites to check them out.

**Webstorm** is the most powerful editor because it has a variety of powerful plugins and features, but I have not used it, because it charges fees. It’s not that I’m reluctant to spend money, but because I think the free Sublime is enough for me. When discussing Webstorm with the interviewer, it doesn’t matter if you haven’t used it, but you must know that it: first, powerful; second, charge fees.

**Sublime** is the editor I use daily, first, it’s free, second, it’s lightweight and efficient, and third, it has many plugins. When using Sublime, make sure to install all kinds of plugins and use them in combination. You can search online for the installation and usage of commonly used plugins for Sublime, as well as its various shortcut keys, and use it personally. I won’t demonstrate them one by one here, the online tutorials are also very foolproof.

**Atom** is an editor produced by GitHub, similar to Sublime, free and rich in plugins, and compared with Sublime, it has a slight fresher style. But I used it a few times and then stopped, because it is slower when it opens and stutters a bit before opening. Of course, overall it is quite good to use, it’s just a matter of personal habit.

**VSCode** is a lightweight (compared to Visual Studio) editor produced by Microsoft. Microsoft is well-known for making good IDEs, which are large and comprehensive, so VSCode also has the various advantages of the above-mentioned Sublime and Atom. However, I also stopped using it after a few times due to personal habit issues (I am not willing to try new things that are not innovative).

To sum up:

- If you want to follow the route of being a big bull, big coffee, high-profile, use Webstorm

- If you follow the ordinary, plebeian, low-key route, use Sublime

- If you’re on the fresh and individual route, use VSCode or Atom

- If you’re being interviewed, it’s best to have one that you’re familiar with, and know a little about the others

Finally, note: Never say that you use Dreamweaver or Notepad++ to write front-end code, you will be despised. If you don’t do .NET, don’t use Visual Studio, and if you don’t do Java, don’t use Eclipse.

## Git

You must have used Git for the projects you have done before, and it must be the command line. If not, you have to review it yourself. If you are familiar with the basic application of Git, you can skip this part. MacOS comes with Git, and Windows needs to install the Git client, you can download it from the Git official website.

> Topic: What are the common Git commands? How to use Git for multi-person collaborative development?

### Common Git commands

First of all, download the latest code through `git clone <project remote address>`, such as `git clone git@git.coding.net:username/project-name.git`, which will download the master branch by default.

Then modify the code, you can see your modification situation through git status during the modification process, and you can view the differences of a single file through `git diff <filename>`.

Finally, submit the modified content to the remote server, do the following operations

```
git add .
git commit -m "xxx"
git push origin master
```

If someone else also submitted the code and you want to synchronize the content submitted by others, execute git pull origin master to do so.

### How to collaborate on development with multiple people

For multi-person collaborative development, you can’t use the master branch, but each developer must pull a separate branch, use `git checkout -b <branchname>`, run git branch to see the names of all local branches.

In your own branch, if you want to synchronize the content of the master branch, you can run git merge master. You can switch branches using `git checkout <branchname>`.

After modifying the content on your own branch, you can submit your own branch to the remote server

```
git add .
git commit -m "xxx"
git push origin <branchname>
```

Finally, after the code is tested without problems, merge the content of your own branch into the master branch, and then submit it to the remote server.

```
git checkout master
git merge <branchname>
git push origin master
```

### About SVN

My attitude towards SVN is same as my attitude towards IE low version browsers, what you need to do is to look up some basic information to understand it a bit. Interviewers might ask about it, but as long as you are familiar with Git operations, the interviewer will not make things difficult for you because you are not familiar with SVN. The premise is that you have to know some basic commands of SVN, and you can check it out online.

However, you have to understand the difference between SVN and Git. SVN is that every step of the operation can not be separated from the server, creating branches, submitting code all need to connect to the server. But Git is different, you can create branches, submit code locally, and then push them to the server all at once. Therefore, Git has all the functions of SVN, but it is much more powerful than SVN. (Git is something invented by Linus, the founder of Linux, so it is also highly praised.)

## Linux Basic Commands

Currently, online servers of Internet companies all use Linux system, and the test environment is definitely using Linux system to ensure consistency with the online environment. And they are all command line, without desktop, can’t use mouse operation. Therefore, it is necessary to master the basic Linux commands. Here are some of the most commonly used Linux commands, I suggest you try them personally under a real Linux system.

Regarding how to get the Linux system, there are two choices: First, install a Linux system in the virtual machine of your computer, such as Ubuntu/CentOS, etc., all of which can be downloaded without money; Second, pay for a Linux virtual machine from cloud providers like Alibaba Cloud. The second one is recommended. Generally, after formal employment, the company will assign you a development machine or a test machine, give you an account and password, and you can log in remotely by yourself.

> Topic: What are the common linux commands?

### Log in

After joining the company, there should exist username and password for you, and once you have them, you can directly log in. Run `ssh name@server` and then enter the password to login.

### Directory Operations

- Create directory `mkdir <directory name>`

- Delete directory `rm <directory name>`

- Locate directory `cd <directory name>`

- View directory file `ls ll`

- Modify directory `name mv <directory name> <new directory name>`

- Copy directory `cp <directory name> <new directory name>`

### File Operations

- Create file `touch <file name> vi <file name>`

- Delete file `rm <file name>`

- Modify file name `mv <file name> <new file name>`

- Copy file `cp <file name> <new file name>`

### File Content Operations

- View file `cat <file name> head <file name> tail <file name>`

- Edit file content `vi <file name>`

- Find file content `grep 'keyword' <file name>`

## Front-end Build Tools

Build tools are an indispensable part of front-end engineering, very important, but have their own speciality in interviews — The interviewer will ask you about the role and purpose of build tools to inquire about your understanding of build tools. As long as you know these, you will not be asked for details. Because, in practice, the real opportunity for writing build tool configuration files is very rare, a project is configured once, and then rarely changed. Moreover, if it’s a widely used framework (such as React, Vue, etc.), there will be ready-made scaffolding tools, one-click creation of a development environment, without manual configuration.

> Topic: Why should front-end use build tools? What problems does it solve?

### What is a Build Tool

“Build” can also be understood as “compile”, which is the process of converting development environment code into runtime environment code. Code in the development environment is for better reading, while code in the runtime environment is for faster execution. Their purposes are different, so the form of code is also different. For example, the JS code written in the development environment needs to be obfuscated and compressed before it can run online, because the code is smaller in this way and it has no impact on code execution. Let’s summarize a few cases where build tools are needed:

- **Processing Modularity**: The modularity syntax of CSS and JS cannot be compatible with the browser at present. Therefore, the established modularity syntax can be used in the development environment, but the modularity syntax needs to be compiled into a form recognizable to the browser by the build tool. For example, use webpack, Rollup, etc. to handle JS modularity.

- **Compile Syntax**: Less and Sass are used to write CSS, and ES6 and TypeScript are used to write JS, etc. These standards can’t be compatible with the browser at present, therefore need to be compiled by build tools, such as using Babel to compile ES6 syntax.

- **Code Compression**: Obfuscate and compress CSS and JS code to make the code smaller and faster to load.

### Introduction to Build Tools

The earliest popularly used build tool was Grunt, soon chased up by Gulp. Gulp is accepted by people because of its simple configuration and efficient performance, and is also one of the build tools recommended by the author. If you are doing some simple JS development, you can consider using it.

If your project is more complicated and developed by many people, then you need to master the artifact of the build tool field — webpack. However, the artifact also has a drawback, which is that the learning cost is relatively high, and you need to take time to study it carefully, not something that can be finished in a few words. Next, we will demonstrate the simplest use of webpack, and for a comprehensive study, you still need to carefully refer to relevant documents, or refer to the tutorial that specifically explains webpack.

### Webpack Demonstration

Next, we will demonstrate the two basic functions of webpack to handle modularity and obfuscate and compress code.

First, you need to install Node.js, if it is not installed, go to the Node.js official website to download and install. After installation, run the following commands to verify that the installation was successful.

```
node -v
npm -v
```

Then, create a new directory, enter the directory, runnpm init, and enter the name, version, description, etc. according to the prompts. After completion, a `package.json` file appears in the directory, which is a JSON file.

Next, install wepback, run `npm i --save-dev webpack`, and wait a few minutes if the network is slow.

Next, write the source code. Create a `src` folder in the directory, and create two files `app.js` and `dt.js` in it, the contents are:

```
// dt.js content
module.exports = {
    getDateNow: function () {
        return Date.now()
    }
}
// app.js content
var dt = require('./dt.js')
alert(dt.getDateNow())
```

Then, go back to the upper directory, create a newindex.htmlfile (this file andsrcare on the same level), and the content is

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>test</title>
</head>
<body>
    <div>test</div>
    
    <script src='./dist/bundle.js'></script>
</body>
</html>
```

Then, write webpack configuration file, create a newwebpack.config.js, the content is

```
const path = require('path');
const webpack = require('webpack');
module.exports = {
  context: path.resolve(__dirname, './src'),
  entry: {
    app: './app.js',
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'bundle.js',
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin({
        compress: {
          //supresses warnings, usually from module minification
          warnings: false
        }
    }),
  ]
};
```

To sum up, the current project’s file directory is:

```
src
  +-- app.js
  +-- dt.js
index.html
package.json
webpack.config.js
```

Next, openpackage.json, and then modify the content ofscriptsto:

```
"scripts": {
    "start": "webpack"
  }
```

Runnpm startin the command line, you can see the results of the compilation, finally openindex.htmlin the browser, and the value ofDate.now()will pop up.

### Summary

In the end, it is emphasized again that deeply understanding the value of the existence of build tools is more meaningful than knowing more configuration code, especially in terms of dealing with interviews.

## Debugging Method

The most tested area of debugging methods is how to capture packets.

> Topic: How to capture data? How to use tools to configure proxy?

For PC-side web pages, we can use the built-in developer tools of Chrome, Firefox and other browsers to view all network requests of web pages to help troubleshoot bugs. This operation of listening and viewing network requests is called packet capture.

For mobile packet capture tools, Charles is recommended under Mac system. First download and install, open. Windows system recommends Fiddler, download, install, and open. The usage of the two is basically the same. The following is an introduction to Charles.

Next, connect the computer installed with Charles and the mobile phone to be captured to the same network (generally the internal network provided by the company, built by professional network engineers), to ensure that the IP section is the same. Then, set the network proxy of the mobile phone (how to set the network proxy for each different mobile phone, there are idiot-style tutorials on the Internet), the proxy IP is the IP of the computer, the proxy port is **8888**. Then, Charles may have a dialog box prompting whether to allow connection to the proxy, select "Allow" here. In this way, the web pages visited by the mobile phone or the requests linked to the network, Charles can listen to.

In the development process, packet capturing tools are often used for proxy. They can proxy the online address to the test environment, both Charles and Fiddler can achieve this function. Let’s take Charles as an example, click on the Tools menu in the menu bar, then click on Map Remote in the submenu, a configuration box will pop up. First of all, tick the Enable Map Remote checkbox, then click on the Add button, add a proxy item. For example, if you want to proxy the online address https://www.aaa.com/api/getuser?name=xxx to the test address http://168.1.1.100:8080/api/getuser?name=xxx.
