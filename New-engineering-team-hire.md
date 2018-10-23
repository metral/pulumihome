# Local setup

### Prerequisites

- Follow [pulumi/home - README](https://github.com/pulumi/home/blob/master/README.md)
  - This will basically help you do the following,
    - Setup `go` toolchain
    - Setup homebrew (for macOS)
    - Setup `node`, `yarn`, `npm` etc.
- After this, you can take specific paths depending on the type of local setup you would like to have.
- Make sure you have access to our AWS account and also setup 2FA.
- Install Docker if you plan to run the service locally at all. 
  - Regardless of whether you plan on running anything locally, if you are going to deploy a stack, you will need Docker to run `docker build`s.

### Run the pulumi-service API and UI locally (recommended)
> This setup is to connect the locally running API service to the storage service, created as part of your personal pulumi stack, in the pulumi AWS account.

- Complete the [setup a new deployment](https://github.com/pulumi/pulumi-service/blob/master/doc/deployment.md#set-up-a-new-deployment) step from the `pulumi-service/doc/deployment.md`.
  - The steps will walk you through running `pulumi-service/scripts/create-stack.sh` to create a stack of your own in AWS.
- After that, for the `pulumi-service` repo...
- In your GOPATH directory, create a folder `pulumi` under `src/github.com`.
- Then clone the `pulumi-service` repo.
  - Running `make ensure` should pull down all necessary dependencies.
  - Running `make build` will build.
  - Running `make lint` will run the linter.
- Now you can run `pulumi-service/scripts/run-service.sh` and the `[Pulumi Console UI](https://github.com/pulumi/pulumi-service/tree/master/cmd/console2)`

### Run the pulumi-service API, UI and also the DB locally
> This setup is to run _everything_ locally.

- Refer to [pulumi-service/doc/running-locally](https://github.com/pulumi/pulumi-service/blob/master/doc/running-locally.md)
- The above guide will walk you through the steps to setup a local database, local object store etc.

### Building the pulumi/pulumi repo
> This repo is the CLI repo. 
- The repo already has a comprehensive README file that talks about local development.
- Along with the specific version for `gometalinter`, which you would have come across in the README inside the `pulumi/home` repo, I have also found that the `pip` version is currently set to `10.0.0` as per `pulumi/build/tool-versions.sh`.
  - You can install the specific version that is mentioned in that script by running (macOS) `sudo easy_install pip==x.y.z`, where x.y.z is the version.

### Troubleshooting

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
  - First, you can run `sudo pip install pipenv`
  - and then
  ```
  export PYTHON_BIN_PATH="$(python -m site --user-base)/bin"
  export PATH="$PATH:$PYTHON_BIN_PATH"
  ```