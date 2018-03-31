Steps for how to deploy the Pulumi Documentation website, housed in [pulumi/docs](https://github.com/pulumi/docs).

Before you do anything though, you need to install the SSH key to connect to the machine. Go to
[Google Drive](https://drive.google.com/drive/u/0/folders/0B0siYR6Ttr5LSXNnMlJ1U256YTg) and
download "pulumi-m2c188q87b.pem" to your "~/.ssh" folder. (Be sure to `chmod 600` it too!)

Then connect to the machine using:

```bash
ssh -i ~/.ssh/pulumi-m2c188q87b.pem ubuntu@ec2-34-214-225-140.us-west-2.compute.amazonaws.com
```

The EC2 instance is provisioned with the `pulumi-bot` SSH key.

## Updating Website Content

To update the website content (e.g. to display a recent update to the [pulumi/docs](https://github.com/pulumi/docs))
repo) just `git pull` on the box and run `make build`.

The `docs` binary is serving directly from a symlink to the `src/github.com/pulumi/docs/_site`
folder, so when the Jekyll build completes everything will be "live".

```bash
cd ~/go/src/github.com/pulumi/docs/
git pull origin production
make build
```

If you make a configuration change to Jekyll, you may need to restart the web server:

```bash
sudo service pulumi-docs restart
```

If for some reason you need to rebuild/restart the docs web server (e.g. a feature change), do the following:

```bash
cd ~/go/src/github.com/pulumi/pulumi-service/
go install ./cmd/docs/

sudo service pulumi-docs restart
```

## Running the Website

The Go `docs` server is run with `systemd` and configured with an environment file.

The environment file should be written to `/home/ubuntu/pulumi-docs/.env` and define:

- `DOCS_ENCRYPTION_KEY`
- `DOCS_AUTHENTICATION_KEY`
- `DOCS_GITHUB_APP_ID`
- `DOCS_GITHUB_APP_SECRET`
- `DOCS_GITHUB_CALLBACK`
- `DOCS_GITHUB_ORG`
- `DOCS_EMAIL_DOMAINS`
- `DOCS_GITHUB_LOGINS`

Details about what these do can be found in `config.go`.

Create a file at `/etc/systemd/system/pulumi-docs.service` with this content:

```ini
# Pulumi docs systemd configuration

[Unit]
Description=Pulumi docs web server

[Service]
Type=simple
User=ubuntu
ExecStart=/home/ubuntu/go/bin/docs -logtostderr
WorkingDirectory=/home/ubuntu/pulumi-docs/
EnvironmentFile=/home/ubuntu/pulumi-docs/.env
Restart=always

[Install]
WantedBy=multi-user.target
```

Start the server with `sudo service pulumi-docs start` and configure it to run on reboot with `sudo systemctl enable pulumi-docs`.

## Viewing website logs

The server uses `systemd` logging. View logs with `journalctl -u pulumi-docs`.

## Initial Setup

Bring down Git repos.

```
mkdir -p go/src/github.com/pulumi/
cd go/src/github.com/pulumi

git clone git@github.com:pulumi/docs.git
git clone git@github.com:pulumi/pulumi-service.git
```

Then install dependencies:

```bash
sudo apt install make

# Ruby
sudo apt install ruby
sudo apt install ruby dev
sudo gem install jekyll bundler

# Go needs a more recent package.
sudo add-apt-repository ppa:gophers/archive
sudo apt update
sudo apt-get install golang-1.9-go
sudo cp /usr/lib/go-1.9/bin/* /usr/bin/

# Add the following to ~/.bashrc
# Then restart bash with `bash`
export GOROOT=$HOME/go
export PATH=$PATH:$GOROOT/bin

go get -u github.com/golang/dep/cmd/dep
go get -u github.com/alecthomas/gometalinter
gometalinter --install

# Node
sudo apt-get install python-software-properties
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install nodejs

sudo apt install npm
sudo npm install --global typedoc
```