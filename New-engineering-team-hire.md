# Your First Week at Pulumi

Welcome to the team!

In your first week, your main deliverable is yourself - making sure you are are setup and ready to contribute to Pulumi!  Here are a few of the things you should target for your first week:

- [ ] Make sure you have access to GSuite, Slack, GitHub, AWS and LastPass.  You'll also eventually need access to Azure, GCP, PagerDuty and more.
- [ ] Make sure you can build, test and dogfood at least https://github.com/pulumi/pulumi and https://github.com/pulumi/pulumi-service.  There are many other repos, but those two are the core homes for our open source offering and backend service respectively.
- [ ] Build a Pulumi application, using the product end-to-end, and send out notes on your experience to the team (a gdoc and a ping on Slack).  This is a great way to get a customer-centric feeling for the whole product before diving into a more focused project.
- [ ] Find out what your first project will be, and get oriented.  Ask who you should talk to on the team to get up to speed on the project.
- [ ] Grab at least 1 issue to fix and send a PR for.  We love it when new employees can merge in their first PR within the first week!  (and ideally, even deliver it to customers in a release!)

# Setting up your dev machine

## Prerequisites

- Follow [pulumi/home - README](https://github.com/pulumi/home/blob/master/README.md)
  - This will basically help you do the following,
    - Setup `go` toolchain
    - Setup homebrew (for macOS)
    - Setup `node`, `yarn`, `npm` etc.
- After this, you can take specific paths depending on the type of local setup you would like to have.
- Make sure you have access to our AWS account and also setup 2FA.
- Install Docker (you will need this for building/running the pulumi service, and for running many Pulumi applications). 
- When installing the `go` dependencies, be sure to check the CI environment scripts to get the specific versions. The README file of each repo does its best to call out the specific versions needed, but when in doubt, check [this](https://github.com/pulumi/scripts/blob/master/ci/install-common-toolchain.sh#L11) script.

## Building the pulumi/pulumi repo
> This repo is the CLI repo. 
- The repo already has a comprehensive [README](https://github.com/pulumi/pulumi/blob/master/README.md) file that talks about local development.
- Along with the specific version for `gometalinter`, which you would have come across in the README inside the `pulumi/home` repo, I have also found that the `pip` version is currently set to `10.0.0` as per `pulumi/build/tool-versions.sh`.
  - You can install the specific version that is mentioned in that script by running (macOS) `sudo easy_install pip==x.y.z`, where x.y.z is the version.

## Building and running the pulumi/pulumi-service repo
> This repo is the Pulumi service hosted at https://app.pulumi.com and https://api.pulumi.com.  Everyone shoud make sure they can build this repo, but not everyone will need to have a personal instance of it (only needed if you are actively developing in the codebase).

### Run the pulumi-service API and UI locally (recommended)
> This setup is to connect the locally running API service to the storage service, created as part of your personal pulumi stack, in the pulumi AWS account.

- Complete the [setup a new deployment](https://github.com/pulumi/pulumi-service/blob/master/doc/deployment.md#set-up-a-new-deployment) step from the `pulumi-service/doc/deployment.md`.
  - The steps will walk you through running `pulumi-service/scripts/create-stack.sh` to create a stack of your own in AWS.
- After that, for the `pulumi-service` repo...
- In your GOPATH directory, create a folder `pulumi` under `src/github.com`.
- Using a terminal, run these commands from the `pulumi-service` repo (you may have already done this as part of setting up your own dev stack):

> You should (and may need to) run `make ensure` occasionally, if any `Go` dependencies have been updated.

  - Running `make ensure` should pull down all necessary dependencies.
  - Running `make build` will build.
  - Running `make lint` will run the linter.
- Register OAuth apps for use with the service running on `localhost` on [GitHub](https://github.com/settings/developers) and [GitLab](https://gitlab.com/profile/applications).
  - You may have already created apps for your dev stack, but these are specifically for your `localhost` and should not be checked-in. See [this](https://github.com/pulumi/pulumi-service/pull/2584#event-2012942101) issue in the `pulumi/pulumi-service` repo.
- Now you can run `pulumi-service/scripts/run-service.sh` and the [`Pulumi Console UI`](https://github.com/pulumi/pulumi-service/tree/master/cmd/console2)

### Run the pulumi-service UI locally using pulumi-test.io

> This slightly simpler setup allows you to run just the UI locally, using the Pulumi test service (at https://app.pulumi-test.io) as an API backend. If the work you're doing is limited to the UI, you may find this setup a little easier to work with.

1. Run `./scripts/run-console frontend`, then navigate to http://localhost:3000/login, bypassing the normal login flow in order to apply an access token that you'll obtain from the test service in the next step.
1. Navigate to https://app.pulumi-test.io and sign in.
1. Browse to Account Settings and click New Access Token.
1. Copy the new token and paste it into the Fake Login view and click Login. You should now be signed in!

### Run the pulumi-service API, UI and also the DB locally
> This setup is to run _everything_ locally.

- Refer to [pulumi-service/doc/running-locally](https://github.com/pulumi/pulumi-service/blob/master/doc/running-locally.md)
- The above guide will walk you through the steps to setup a local database, local object store etc.

# Troubleshooting

##### (macOS) Error from homebrew while installing `awscli`

If you get an error like this:
```Error: An unexpected error occurred during the `brew link` step
The formula built, but is not symlinked into /usr/local
Permission denied @ dir_s_mkdir - /usr/local/Frameworks
Error: Permission denied @ dir_s_mkdir - /usr/local/Frameworks```

Then, run the following commands:
```
sudo mkdir /usr/local/Frameworks
sudo chown $(whoami):admin /usr/local/Frameworks/
```
and then re-run `brew install awscli` again, so that it can complete the linking step successfully.

##### (macOS) Error during `make ensure` in the `pulumi/pulumi` repo
- If you run into the following error when you run `make ensure` in the repo,
  ```error in pipenv setup command: 'install_requires' must be a string or list of strings containing valid project/version requirement specifiers; Expected version spec in requests[security];python_version<"2.7" at ;python_version<"2.7"```
  - This is because the version of the python package called `setuptools` isn't up-to-date. So run, `sudo easy_install -U setuptools` to fix this.
- If you also run into an error about the `pipenv: command not found`, then your installation of `pipenv` may not be in the `PATH`. So add it to your path with this:
  - First, run `sudo pip3 install pipenv`
    - Note that we are using `pip3`, since we use Python3 in our Python SDK.
    - This means, you must have installed `python3` either using `brew` or downloaded the Python3 installer from the official Python site.
  - Installing via the official Python3 installer means, it'll update your bash profile and add Python3 to your `Path` automatically.