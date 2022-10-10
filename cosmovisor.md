# Using Cosmovisor with und: Quick Start

`cosmovisor` can be used for all node types, not just validator nodes.

See [Cosmos SDK's Cosmovidor Docs](https://docs.cosmos.network/v0.44/run-node/cosmovisor.html) for full documentation.

This is a quick-start guide specifically for getting `und` up and running using `cosmovisor` on **MainNet**. It is 
recommended that node operators fully read and understand the official documentation above before proceeding.

**Note**: The above documentation states that `go get github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor` should be run
to install `cosmovisor`. However, this is not currently possible due to [issue 11305](https://github.com/cosmos/cosmos-sdk/issues/11305).
As such, this guide directly downloads the `cosmovisor` binary from the Cosmos SDK repo. If you prefer to install via `go`,
you will need to install `go` v1.18, `make`, `git`, and standard build tools such as `g++`, and build `cosmovisor` v1.2.0 from source,
ignoring step 1 below.

This guide assumes the `und` binary is running with `systemd` and that the user logging in via SSH is a `sudoer`. It also
assumes that the `.und_mainchain` home directory is the default `$HOME/.und_mainchain`.

## 1. Download & install cosmovisor

```bash
mkdir $HOME/tmp && cd $HOME/tmp
wget https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.2.0/cosmovisor-v1.2.0-linux-amd64.tar.gz
tar -zxvf cosmovisor-v1.2.0-linux-amd64.tar.gz
sudo mv cosmovisor /usr/local/bin/cosmovisor
```

**Note**: Installing using `go install` as per the official guide will install `cosmovisor` in `$GOPATH/bin` (usually
`$HOME/go/bin`) not `/usr/local/bin`.

## 2. Set up cosmovisor dirs

```bash
mkdir -p $HOME/.und_mainchain/cosmovisor/genesis/bin
cd $HOME/tmp
wget https://github.com/unification-com/mainchain/releases/download/1.5.1/und_v1.5.1_linux_x86_64.tar.gz
tar -zxvf und_v1.5.1_linux_x86_64.tar.gz
mv und $HOME/.und_mainchain/cosmovisor/genesis/bin
```

Check the version:

```bash
$HOME/.und_mainchain/cosmovisor/genesis/bin/und version --log_level=""
1.5.1
```

Create a `UND_COSMOVISOR_ENV` env file for `systemd`:

```bash
nano $HOME/.und_mainchain/cosmovisor/UND_COSMOVISOR_ENV
```

Add:

```bash
DAEMON_NAME=und
DAEMON_HOME=$HOME/.und_mainchain
DAEMON_RESTART_AFTER_UPGRADE=true
```

Optionally add a value for `DAEMON_RESTART_DELAY` (must be in seconds, e.g. `5s`)

```bash
DAEMON_RESTART_DELAY=5s
```

See https://github.com/cosmos/cosmos-sdk/blob/main/cosmovisor/README.md#command-line-arguments-and-environment-variables for
other environment variables.

## 3. Edit systemd service file

Stop the service

```bash
sudo systemctl stop und
```

Open the service file in a text editor

```bash
sudo nano /etc/systemd/system/und.service
```

Add the `EnvironmentFile` directive to the `[Service]` section as follows:

```bash
EnvironmentFile=/absolute/full/path/to/.und_mainchain/cosmovisor/UND_COSMOVISOR_ENV
```

Modify `/absolute/full/path/to/.und_mainchain/cosmovisor/UND_COSMOVISOR_ENV` accordingly.

Edit the `ExecStart` directive, replacing `/usr/local/bin/und` with `/usr/local/bin/cosmovisor run`, for example:

```bash
ExecStart=/usr/local/bin/cosmovisor run start
```

If you have installed via `go install` or built/installed from source, the path might be `/home/centos/go/bin/cosmovisor`

**Note**: Any flags for `und` can be passed the same as previously, for example:

```bash
ExecStart=/usr/local/bin/cosmovisor run start --home /absolute/full/path/to/.und_mainchain
```

## 4. restart the und service

Reload `systemd` and start the `und` service

```bash
sudo systemctl daemon-reload
sudo systemctl start und
```

Check:

```bash
sudo journalctl -u und -f
```

The logs should be prefixed with `cosmovisor[PID]`
