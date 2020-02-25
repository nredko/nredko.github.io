---
title: SSH port forwarding in vscode (Visual Studio Code)
tags: [ssh, vscode, port forwarding]
style: fill
color: primary
description: VS Code has great feature for remote debugging, but configuring port forwarding is not obvious
---

## VSCode remote debugging

It's really great feature - you can have your code on remopte server, use git repo on remote server, run debug on remote server and so on, having only windows of IDE on yuor local computer.

First steps to configure vscode for remote development on Windows:

https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse 

https://code.visualstudio.com/blogs/2019/07/25/remote-ssh 


## Port forwarding

If you run Java, Python, NodeJs, Go or whatever on remote Linux server, in many cases it is the _server_ which you're trying to develop. And if for some reason you server expose some TCP port which is not available from external network (while you have exposed SSH for connection), you need to see this port _forwarded_ to your local computer.

See short description how this can be done within the same SSH connection which you use to connect to console or file system (via SCP or SFTP):

https://www.ssh.com/ssh/tunneling/example

## Ususal errors in vscode

If you'll setup your connection string as
```bash
ssh -A -L 8080:localhost:8080 user@myserver.address.com
```
you'll get error like `The process tried to write to a nonexistent pipe ...`

Search over internet doesn't give you useful answers.


## Solution

After configuring connection string as `ssh -A -L 8080:localhost:8080 user@myserver.address.com` open yous ssh config file (usually `C:\Users\%UserName%\.ssh\config` for local user's configuration) you should change 
the string `LocalForward 8080:localhost:8080` to `LocalForward localhost:8080 localhost:8080`. 

*That's it!*

Thanks _black_sheep_ for an [answer](https://superuser.com/questions/722473/ssh-config-missing-target-argument/722474)
