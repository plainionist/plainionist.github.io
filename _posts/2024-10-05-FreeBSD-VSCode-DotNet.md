---
layout: post
title: "C# development in a FreeBSD jail?"
description: |
    FreeBSD jails allow running an almost full featured linux on FreeBSD.
    Jails even support running X11 apps. 
    But can we use it to do C# development using VSCode?
tags: [technologies]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

While VSCode and DotNet generally works on FreeBSD, the "C# Dev Kit" does not due to the following issue:
[https://github.com/microsoft/vscode-dotnettools/issues/1388]

Knowing that FreeBSD jails support running an almost full features linux system and also X11 apps,
I was curious whether it would be possible to run VSCode including "C# Dev Kit" in a jail.

Here is what I learned.

<!--more-->

# Setting up the Jail

For the basic linux jail setup I mostly followed this great article:

[https://forums.freebsd.org/threads/setting-up-a-debian-linux-jail-on-freebsd.68434/]

After several attempts, this is the jail configuration I finally used for my experiments

```
devuan {
    host.hostname = "${name}.jail";
    interface = re0;
    ip4.addr= 192.168.0.113;
    allow.raw_sockets=1;

    allow.sysvipc;

    path = "/opt/jails/${name}";

    exec.start = "/etc/init.d/rc 3";
    exec.poststart = "jexec ${name} mkdir /tmp/.X11-unix; jexec ${name} chmod 777 /tmp/.X11-unix; mount_nullfs /tmp/.X11-unix /opt/jails/${name}/tmp/.X11-unix";
    exec.stop = "/etc/init.d/rc 0";
    exec.poststop = "umount -f /opt/jails/${name}/tmp/.X11-unix; umount -f -a -F /opt/jails/${name}.fstab";

    persist;

    allow.mount;
    allow.mount.procfs;
    allow.mount.linsysfs;
    allow.mount.tmpfs;
    allow.mount.devfs;

    allow.mlock;

    mount.fstab = "/opt/jails/${name}.fstab";
} 
```

To simplify the network setup, I decided for direct network access.
I also learned that "devfs" needs to be mounted via fstab to also support "/dev/shm".

I used "exec.poststart" and "exec.poststop" to automatically setup unix socket for X11
and also to unmount the fstab entries as this didn't happen automatically when shutting down the jail.

It also took me several attempts to finally come up with this fstab.

```
# Dev       Mountpoint                  FS          Options           Dump/Check

devfs       /opt/jails/devuan/dev       devfs       rw,late             0     0
tmpfs       /opt/jails/devuan/dev/shm   tmpfs       rw,late,mode=1777   0     0
fdescfs     /opt/jails/devuan/dev/fd    fdescfs     rw,late,linrdlnk    0     0

linprocfs   /opt/jails/devuan/proc      linprocfs   rw,late             0     0
linsysfs    /opt/jails/devuan/sys       linsysfs    rw,late             0     0
tmpfs       /opt/jails/devuan/tmp       tmpfs       rw,late,mode=1777   0     0
```

# Setting up X11

For setting up X11 I mostly followed this article: [https://wiki.freebsd.org/JailingGUIApplications]

I set up a non root user as follows:

```bash
adduser dev1
usermod -aG video,audio,input,plugdev dev1
```

Additionally to the Unix socket setup (see jail.conf) I ran ``xhost +`` on the host to allow X11 access.

I also had to copy the ".Xauthority" from my home directory at the host system
and then added the following lines to "/home/dev1/.profile"

```bash
export DISPLAY=:0.0
export XAUTHORITY=/home/dev1/.Xauthority 
```

To run X11 apps it is important to enter the jail without root permission:

```bash
jexec devuan su - dev1
```

I tried the X11 setup by running "mousepad" which, apart from some warnings, worked pretty well!

```
(mousepad:3416): dconf-WARNING **: 10:43:01.335: failed to commit changes to dconf: Could not connect: Connection refused
```

# Setting up VSCode

I downloaded the "deb" package from the VSCode home page and used ``apt install`` to install it.
The first attempt failed. I had to adjust the apt cache by adding the following line to "/etc/apt/apt.conf":

```
APT::Cache-Start "104857600";
```

After running ``apt-get update``, I succeeded installing VSCode.

The first attempt to run VSCode failed:

```bash
dev1@devuan:~$ code --verbose
/usr/share/code/code: error while loading shared libraries: libffmpeg.so: cannot open shared object file: No such file or directory
[3476:0922/104437.119547:FATAL:zygote_host_impl_linux.cc(198)] Check failed: . : Invalid argument (22)
```

To fix these issues I installed ffmpeg and added the following configuration for the dynamic linker:

```bash
echo "/usr/share/code" > /etc/ld.so.conf.d/vscode.conf
```
During my research I found hints to try running VSCode using "--no-sandbox" option.

## Second attempt

But also the second attempt failed:

```bash
dev1@devuan:/usr/share/code$ code --no-sandbox --verbose
[4701:0922/105600.213686:ERROR:file_path_watcher_inotify.cc(890)] Failed to read /proc/sys/fs/inotify/max_user_watches
[4700:0922/105600.213687:ERROR:file_path_watcher_inotify.cc(890)] Failed to read /proc/sys/fs/inotify/max_user_watches
[4699:0922/105600.214927:ERROR:bus.cc(407)] Failed to connect to the bus: Failed to connect to socket /run/dbus/system_bus_socket: No such file or directory
[4699:0922/105600.452318:ERROR:bus.cc(407)] Failed to connect to the bus: Failed to connect to socket /run/dbus/system_bus_socket: No such file or directory
[4699:0922/105600.452390:ERROR:bus.cc(407)] Failed to connect to the bus: Failed to connect to socket /run/dbus/system_bus_socket: No such file or directory
[4699:0922/105600.453678:ERROR:bus.cc(407)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
[4699:0922/105600.453740:ERROR:bus.cc(407)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
[4699:0922/105600.453766:ERROR:bus.cc(407)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
[4699:0922/105600.453988:ERROR:file_path_watcher_inotify.cc(337)] inotify_init() failed: Function not implemented (38)
[4699:0922/105600.454940:ERROR:bus.cc(407)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
[4699:0922/105600.455001:ERROR:platform_shared_memory_region_posix.cc(214)] Creating shared memory in /dev/shm/.org.chromium.Chromium.x7kIrq failed: No such file or directory (2)
[4699:0922/105600.455029:ERROR:platform_shared_memory_region_posix.cc(217)] Unable to access(W_OK|X_OK) /dev/shm: No such file or directory (2)
[4699:0922/105600.455043:FATAL:platform_shared_memory_region_posix.cc(219)] This is frequently caused by incorrect permissions on /dev/shm.  Try 'sudo chmod 1777 /dev/shm' to fix.
```

At this point I updated the fstab to support "/dev/shm".

To make "DBus" running I removed the following check from "/etc/init.d/dbus":

```bash
if ! mountpoint -q /proc/ ; then
    log_failure_msg "Can't start $DESC - /proc is not mounted"
    return
fi
```

## Third attempt

Finally, I was able to launch VSCode with the following command:

```
code --no-sandbox --disable-file-watcher --verbose
```

It worked, but the terminal was still flooded with warnings and errors.

# Installing DotNet SDK

I followed Microsoft's guide to install the .NET SDK:
https://learn.microsoft.com/en-us/dotnet/core/install/linux-debian

But ``apt-get update`` caused the following error

```
W: Failed to fetch https://packages.microsoft.com/debian/12/prod/dists/bookworm/InRelease  Bad header line Bad header data [IP: 13.107.246.45 443]
```

To fix this problem I ran:

```bash
curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg
rm -rf /var/lib/apt/lists/* 
apt-get update
```

Then the installation succeeded:

```bash
apt-get install -y dotnet-sdk-8.0
```

# Installing "C# Dev Kit" VSCode extension

The installation of "C# Dev Kit" extension finished without any visible error from the extension manager but the in the logs i found:

```
The .NET Core SDK cannot be located: Error running dotnet --info: Error: Command failed: dotnet --info
Failed to create CoreCLR, HRESULT: 0x8007FF02
```

I continued my research and found the following issue: https://github.com/dotnet/runtime/issues/44417

After having added "allow.mlock" to the jail configuration I could successfully run 

```bash
dotnet new
```

but when I tried

```bash
dotnet new console
```

I got the following error:

```
The template "Console App" was created successfully.

Processing post-creation actions...
Restoring /home/dev1/ws/test/test.csproj:
MSBUILD : error : This is an unhandled exception in MSBuild -- PLEASE UPVOTE AN EXISTING ISSUE OR FILE A NEW ONE AT https://aka.ms/msbuild/unhandled
MSBUILD : error :     System.Threading.ThreadStateException: Unable to set thread priority.                                                                                                                                            
MSBUILD : error :    at System.Threading.Thread.SetPriorityNative(Int32 priority)                                                                                                                                                      
MSBUILD : error :    at System.Threading.Thread.set_Priority(ThreadPriority value)                                                                                                                                                     
MSBUILD : error :    at Microsoft.Build.BackEnd.RequestBuilder.SetCommonWorkerThreadParameters()                                                                                                                                       
MSBUILD : error :    at Microsoft.Build.BackEnd.RequestBuilder.RequestThreadProc(Boolean setThreadParameters)                                                                                                                          
Restore failed.                                                                                                                                                                                                                        
Post action failed.
Manual instructions: Run 'dotnet restore'
```

Despite extensive research, I couldn’t find a workaround for this issue.
At this point I stopped the experiment.

# Conclusion

It seems FreeBSD jails are not suitable for C# development. 
I hope Microsoft fixes the issue in the "C# Dev Kit" soon.
In the mean time I will start a new experiment: Arch Linux in bhyve

