# Executable for `npiperelay`
This is the Windows executable of the [`npiperelay`](https://github.com/jstarks/npiperelay) project, which does not have a pre-built image available.

# Usage

For my use case, I wanted to use the OpenSSH agent in WSL and Docker. `npiperelay` was pivotal in implementing this functionality. In my case, I made a bash script that did the following:

```bash
#!/bin/bash

# Create a WSL socket for the openSSH agent. This allows using the SSH credentials
# from Windows in WSL and Docker if the stream given by /tmp/ssh-agent.sock is mounted.
# Note that this will also use 1Password managed credentials.
export SSH_AUTH_SOCK=/tmp/ssh-agent.sock

if [ ! -e "$SSH_AUTH_SOCK" ]; then
        exec socat \
                UNIX-LISTEN:${SSH_AUTH_SOCK},umask=007,fork \
                EXEC:'npiperelay.exe -ep -s //./pipe/openssh-ssh-agent',nofork &
fi
```

When this script is run, it creates a symbol for `SSH_AUTH_SOCK`. If this file doesn't exist, then it creates a socket that listens to `SSH_AUTH_SOCK` and pipes it to the OpenSSH agent.

# How I built it

These instructions were based on the npiperelay issue [here](https://github.com/jstarks/npiperelay/issues/25). To build it, I installed Go, then executed the following in WSL.

```bash
GOOS=windows go install github.com/jstarks/npiperelay@latest
```

This deposits a file in `$HOME/go/bin/windows_amd64` called `npiperelay.exe`. To make it a bit more accessible, I create a link in my `/usr/local/bin` directory.

```bash
sudo ln -s $HOME/go/bin/windows_amd64/npiperelay.exe /usr/local/bin/npiperelay.exe
```

