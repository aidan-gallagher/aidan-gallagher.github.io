---
title: "Remote Debugging Using GDB and VSCode"
excerpt: "Notes on using gdb and VSCode to remotely debug a daemon on a Debian system."
categories:
  - Notes
tags:
  - Debugging
---

# Introduction

Remote debugging is debugging a program that doesn't run on you local machine. This is done by connecting the remotely running program to your local development environment.

If you have the option it is usually better to debug your program locally. However, sometimes your program may behave erroneously when integrated within a system and it's easier to debug the program remotely than it is to try and replicate those conditions locally.

The following steps are for debugging an operating system daemon written in C/C++ on a Debian system; for other usecases you may have to tweak the steps.

# Steps

## Prepare the Program

1. Package the program with debug symbols and without optimisation.  
This should be achieved by prepending `DEB_BUILD_OPTIONS=debug,nostrip,noopt` to `dpkg-buildpackage`. 
```
DEB_BUILD_OPTIONS=debug,nostrip,noopt dpkg-buildpackage 
```

If the package's debian/rules isn't setup correctly to handle these options then you may have to edit the build system manually (e.g. meson.build file) to set the debug on & optimisation off.


2. Copy the package to the remote machine's home directory.
```
scp ../<my_deb_pacakge>.deb <management_ip_address>:~
```

## Prepare the Remote Machine

1. Log in to the remote machine.
```
ssh <management_ip_address>
```

2. Install GDB server.
```
sudo apt install gdbserver
```

3. Install the package.
```
sudo dpkg -i ./<my_deb_package>.deb
```

4. Restart the program.
```
sudo systemctl restart <my_program>
```

5. Attach GDB server to the running program.
```
sudo gdbserver --attach <management_ip_address>:2345 `pidof <my_program>`
```

## Start Debugging on Local Machine

1. Debug using CLI to verifiy everything so far is working.
``` 
gdb <my_program>
...
target remote <management_ip_address>:2345
```

2. Add debug configuration to `.vscode/launch.json`.
```
{
  "configurations": [
    {
      "name": "Attach to running <my_program>",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceRoot}/build/src/<my_program>",
      "miDebuggerServerAddress": "${input:remote_ip_address}:2345",
      "cwd": "${workspaceRoot}",
    }
  ],
  "inputs": [
    {
      "id": "remote_ip_address",
      "type":"promptString",
      "description": "Management IP address of remote machine to debug.",
      "default": "<management_ip_address>",  # Can change to current address so have to retype every time.
    },
  ]
}
```

4. Start debugging, pause the program and set breakpoints.

5. Prepend any GDB commands in the VSCode debug console with `-exec`.
```
-exec show scheduler-locking 
```



# Notes

## Port numbers
The port number for the GDB server can be any value provided
* It's not already in use.
* You use the same number for the GDB client.

## Apt sources
Some embedded Debian devices have their apt sources removed. To add them back in use the following command.  
```
echo "deb http://deb.debian.org/debian bullseye main contrib non-free
deb-src http://deb.debian.org/debian bullseye main contrib non-free
  
deb http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
deb-src http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
  
deb http://deb.debian.org/debian bullseye-updates main contrib non-free
deb-src http://deb.debian.org/debian bullseye-updates main contrib non-free" | sudo tee --append /etc/apt/sources.list > /dev/null
 
sudo apt update
```
Note: You may have to change the version `bullseye` if you aren't using Debian 11.

## Timeout Errors
When you pause your program during the debugging it will no longer be able to respond to other components. Therefore you should not be surprised to see lots of timeout errors in the system log.

