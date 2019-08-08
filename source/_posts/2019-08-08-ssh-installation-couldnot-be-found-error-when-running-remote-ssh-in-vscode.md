title: "An SSH installation couldn't be found" error when running "Remote - SSH" in Visual Studio Code
categories:
- Visual Studio Code
---
I ran into an error when running "Remote - SSH" in Visual Studio Code on Windows. The error shows that an SSH installation couldn't be found, but I've installed the git client. Finally I found some useful info in the doc:

> VS Code will look for the `ssh` command in the PATH. Failing that, on Windows it will attempt to find `ssh.exe` in the default Git for Windows install path. You can also specifically tell VS Code where to find the SSH client by adding the `remote.SSH.path` property to `settings.json`.

My git client was not installed in the default path, thus VS Code couldn't find the `ssh` command. So I added the `remote.SSH.path` property to `settings.json` and problem solved.
