Before setting up your machine, no matter the operating system you will need to configure various package managers before you'll be able to build and test many of our projects.

# JavaScript (NPM)

For our JavaScript repos, we use NPM for package management, and publish private packages inside of our own `@pulumi` organization.  To access them, you will _either_ need an https://npmjs.com/ account _or_ you will need to configure your NPM client to use our private proxy.  The former is preferred, but the latter is relatively easy.

To configure your client to use our proxy, add this to `~/.npmrc`:

```
@pulumi:registry=https://npmjs.pulumi.com/
//npmjs.pulumi.com/:_authToken=${PULUMI_ACCESS_TOKEN}
```

where `PULUMI_ACCESS_TOKEN` is the token listed on your https://beta.pulumi.com/account page.

# Python (PyPI)

For our Python repos, we use our private PyPI server for package management.  This is temporary, until we are public, since PyPI doesn't have the equivalent of private packages.  To use it, you'll need to add the following to your `pip.conf` file:

```
[global]
extra-index-url = https://${PULUMI_ACCESS_TOKEN}@pypi.pulumi.com/simple
```

where `PULUMI_ACCESS_TOKEN` is the token listed on your https://beta.pulumi.com/account page.

Note that the location of this file is [different on different operating systems](https://pip.pypa.io/en/stable/user_guide/#config-file):

* On Unix: `$HOME/.config/pip/pip.conf`
* On macOS: `$HOME/Library/Application Support/pip/pip.conf`
* On Windows: `%APPDATA%\pip\pip.ini`

You can set a custom path location for this config file using the environment variable `PIP_CONFIG_FILE`.