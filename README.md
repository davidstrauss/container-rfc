Standard Container RFC
--
Current status: _WIP draft_. Please contribute. Not complete at all. 

A vendor neutral format for Linux container images and runtime. This document specifies services the runtime needs to expose to the container, as well as the format the runtime expects to be able run the specified container image. 

## How to Contribute

Please fork, then issue a pull request. We will discuss in the PRs. 

## Goals

* __Portability__, Given a standard container, many runtimes can run it
* __Simple__, should be already implemented or trivial to support in existing runtimes


## Image Format

The basic format is a root filesystem containing all the dependencies needed to run an application. A basic chroot should be a sufficient test to see if your filesystem is setup correctly. 

For the purposes of this document, we are ignoring transport or packaging. The runtime simply needs a root filesystem to start from. 


### Required Files

These files do not need to have any contents, but should be provided either by the runtime or the contents of the file.

#### /etc/os-release

[Operating system identification](http://www.freedesktop.org/software/systemd/man/os-release.html). Can be a blank file, but the runtime can inspect it before starting. Here are a [few comments](http://0pointer.de/blog/projects/os-release.html) on why it is important. 

#### /etc/resolv.conf

If provided by the container, the runtime must respect it. If not provided, the runtime can inject or add it to the container. 

#### Example RFC minimal container

What is the smallest container we can make to test this?

### Runtime Support

#### Entry point

The runtime should provide a mechanism to enter the container through an arbitrary executable. If not specified, runtime should default to /sbin/init. Otherwise fail. 

##### Environment Variables

Runtime should support configuring environment variables via a file. 

#### Networking types

The runtime should support three types of networking. 

##### Host networking

The hosts network adapter is attached directly to the container. No networking namespace. 

##### Private networking

A network namespace just for the container, no external networking. 

##### Virtual/NAT networking

Optionally, the runtime can provide its own networking virtualization layer. This should be exposed to the host as a nic. 

#### Namespace support

Runtime should support full process, filesystem, user, network namespaces. 

#### Cgroup support

Runtime should support memory limiting, memory with swap limiting, disk IO limits, and CPU thresholds. 


### Runtime provides

#### Unique ID

Once running, the runtime should provide a globally unique ID for the running container. 

## Questions?
* Where does socket activation come in? does it?
* Runtime able to specify receiving port, and map to internal port or over a file descriptor


## Support matrix

|Runtime Support|Docker|[systemd+nspawn](http://www.freedesktop.org/software/systemd/man/systemd-nspawn.html)|native LXC|libvirt-lxc|lmctfy|OpenVZ|
|----|----|----|---|---|---|---|
|/etc/resolv.conf|yes|yes|-|-|-|-|
|/etc/os-release|no|yes|-|-|-|-|
|/usr/sbin/init|no|yes|-|-|-|-|
|EnvironmentFile|no|yes|-|-|-|-|
|NAT networking|yes|no|-|-|-|-|
|Private networking|yes|yes|-|-|-|-|
|Host networking|no|yes|-|-|-|-|
|Socket activation|no|yes|no|yes|no|no|
|Unique container id|yes|yes|-|-|-|-|
|Network namespaces|yes|yes|-|-|-|-|
|Filesystem namespaces|yes|yes|-|-|-|-|
|Process namespaces|yes|yes|-|-|-|-|
|User namespaces|no|no|-|-|-|-|
|Cgroups|yes|yes|-|-|-|-|



## References

Some places of inspiration for this.

* [systemd-nspawn](http://www.freedesktop.org/software/systemd/man/systemd-nspawn.html)

* docker inspect 

```
[{
    "ID": "3c85249977e132676c68f2dd03c5e7ad556755cdfc9188ce8dd15a17b5f66e44",
    "Created": "2013-11-09T05:13:10.109281735Z",
    "Path": "/bin/sh",
    "Args": [
        "-c",
        "/usr/sbin/nginx"
    ],
    "Config": {
        "Hostname": "3c85249977e1",
        "Domainname": "",
        "User": "",
        "Memory": 0,
        "MemorySwap": 0,
        "CpuShares": 0,
        "AttachStdin": false,
        "AttachStdout": false,
        "AttachStderr": false,
        "PortSpecs": null,
        "ExposedPorts": {
            "9001/tcp": {}
        },
        "Tty": false,
        "OpenStdin": false,
        "StdinOnce": false,
        "Env": [
            "HOME=/",
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Cmd": [],
        "Dns": null,
        "Image": "162.209.1.102:49193/pages",
        "Volumes": {},
        "VolumesFrom": "",
        "WorkingDir": "",
        "Entrypoint": [
            "/bin/sh",
            "-c",
            "/usr/sbin/nginx"
        ],
        "NetworkDisabled": false,
        "Privileged": false
    },
    "State": {
        "Running": true,
        "Pid": 9363,
        "ExitCode": 0,
        "StartedAt": "2013-11-09T05:13:10.329784251Z",
        "FinishedAt": "0001-01-01T00:00:00Z",
        "Ghost": false
    },
    "Image": "b10492dd794a83c3be908bce78c17783b667130aa529ed9346db82b8fc637671",
    "NetworkSettings": {
        "IPAddress": "172.17.0.200",
        "IPPrefixLen": 16,
        "Gateway": "172.17.42.1",
        "Bridge": "docker0",
        "PortMapping": null,
        "Ports": {
            "9001/tcp": [
                {
                    "HostIp": "0.0.0.0",
                    "HostPort": "49195"
                }
            ]
        }
    },
    "SysInitPath": "/usr/libexec/docker/dockerinit",
    "ResolvConfPath": "/etc/resolv.conf",
    "HostnamePath": "/var/lib/docker/containers/3c85249977e132676c68f2dd03c5e7ad556755cdfc9188ce8dd15a17b5f66e44/hostname",
    "HostsPath": "/var/lib/docker/containers/3c85249977e132676c68f2dd03c5e7ad556755cdfc9188ce8dd15a17b5f66e44/hosts",
    "Name": "/blue_lion6",
    "Volumes": {},
    "VolumesRW": {}
}]
```

* [A gist for required stuff docker needs on a minimal container](https://gist.github.com/jpetazzo/b932fb0c753e69c73d31/raw)
