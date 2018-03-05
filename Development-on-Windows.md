# Setting up a Development Environment on Windows

## Warning

We rely on a combination of approaches, including cross-compliation, for Windows. For our go code, we cross compile by setting `GOOS`. For our JavaScript/TypeScript code, no cross compilation is, in general needed. However, we have a current exception with `pulumi/pulumi` that needs to build a native node module for our language host to serialize JavaScript functions. So we maintain enough support in pulumi/pulumi to build on Windows. 

We've experimented with using the Linux Subsystem for Windows to support our existing *nix based infrastructure on Windows, but it wasn't a good experience. If you want to live in Windows day to day, you'll want to have a VM you can connect to (either via SSH or the console) and for your development.

## Install pre-requisites

This list was current as of 11/7/2017. 

In general, moving to newer versions of the tools should be okay, with the exception of Node. We **must** use 6.10.2. Unless specified for a specific tool, just accepting the defaults when you install is fine.

1. Download [Visual Studio 2017 Community](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15) and check the Desktop Development with C++ feature.
2. Download [Go 1.9.2](https://redirector.gvt1.com/edgedl/go/go1.9.2.windows-amd64.msi) and install it.
3. Download [NodeJS 6.10.2](http://nodejs.org/dist/v6.10.2/node-v6.10.2-x64.msi) and install it. You must pick **EXACTLY** this version on Node
4. Download [Python 2.7.14](https://www.python.org/ftp/python/2.7.14/python-2.7.14.amd64.msi) and install it. You'll want to select the "Add python.exe to Path" feature when you have the option to customize your install
5. Download [Yarn](https://yarnpkg.com/latest.msi) and install it.
6. Download [AWS CLI](https://s3.amazonaws.com/aws-cli/AWSCLI64.msi) and install it.
7. Download [7-Zip](http://www.7-zip.org/a/7z1604-x64.msi) and install it. You'll also need to ensure the folder you installed it to is on the `%PATH%` as some of our automation requires it. As a test, just try running `7z` from a command window and ensure 7-Zip gets run.
8. Download [Git For Windows](https://github.com/git-for-windows/git/releases/download/v2.15.0.windows.1/Git-2.15.0-64-bit.exe) and install it. One nice feature of Git for Windows is it can handle generating an access token for your GitHub account so you can clone our private repositories without having to worry about SSH Keys and the like.
9. Install `dep`, using go get: `go get -u github.com/golang/dep/cmd/dep`

## Configure go

By default, go will use `%USERPROFILE%\go` as your default `GOPATH`. This is a fine default, but if you'd like to store your source code somewhere else, you'll want to configure that now. Either way, you'll also want to add the bin folder in your `GOPATH` to the system path so commands like `dep` work on the console and when we `go install` pulumi it will be on the path as well.

## Clone source

Today, we only support building `pulumi/pulumi` on Windows. In addition to the `pulumi/pulumi` repository, we need to clone `pulumi/home` which has some shared infrastructure. If you've explicitly set a GOPATH environment variable, you'll want to clone your sources there. The default is `%USERPROFILE%\go`. Inside your `src\github.com\pulumi` folder, clone the two repositories we care about:

```
C:\Users\Matt Ellis\go\src\github.com\pulumi> git clone https://github.com/pulumi/pulumi.git
C:\Users\Matt Ellis\go\src\github.com\pulumi> git clone https://github.com/pulumi/home.git
```

The initial clone should ask you to log into GitHub (and handle 2FA).

## Configure AWS

Our build pulls down some prebuilt binaries from S3, so you'll need to ensure you are logged into S3. Installing the AWSSDK above should have put `aws` on your command line, so you can run `aws configure`. Do so, and set your AWS Access Key ID and AWS Secret Access Key.

## Build/Test

Launch a "Developer Command Prompt for VS 2017" window, navigate to your `pulumi/pulumi` repository and run `msbuild`. The initial build will take some time to download bout our custom `node.exe` we use at runtime and the sources for Node 6.10.2 so we can build our native plugin. If you want to run the integration tests, build the `IntegrationTest` target. The other two interesting targets are `AppVeyorPush` and `AppVeyorPullRequest` which are the targets that are run as part of CI.