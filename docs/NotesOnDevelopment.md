# Iamus: Notes On Development

This document attempts to describe the environment you need to build
and configure in order to enhance and debug a Iamus metaverse server.

Building and running this metaverse server is tricky because there
are several applications and services that must link together:
- metaverse-server: presents API for domain and account management;
- ice-server: provides linkage between Interface's for firewall transversal and streaming;
- domain-server: hosts the "land" and avatars that make up a region of the metaverse;
- Interface: the "user interface" that talks to the above services and presents the user's view of the metaverse;

All four of this services run as separate processes, can run on different computers,
and all have to properly link to each other.

Below is described building the different servers.
Especially check out the [Modifications](#Modification) section for the changes that must
be made to the sources to get all the URLs correct.
The original sources from High Fidelity contained many, many URLs as in-source
constants which need changing to run a different metaverse-server instance.

## Metaverse-Server

I do my development of the metaverse-server on a Linux box
using NodeJS version 14.
If I am running the server as a separate service, I build a [Docker] image
and run it in a [DigitalOcean] droplet.
Refer to [Running Docker Image] for the latter case. What follows is
my development setup.

## Building and Running Ice-Server and Domain-Server

The ice-server and domain-server are part of the [Vircadia] project.
For the ice-server and domain-server, I've been using the
[vircadia-builder] project to build a properly configured
version of these services.

I use [vircadia-builder] to build all the pieces, then I modify
the sources, and then do a rebuild to create images with the modifications.
The process is:

```sh
cd
git clone https://github.com/vircadia/vircadia-builder.git
cd vircadia-builder
./vircadia-builder --tag=master --build=domain-server,ice-server,assignment-client
cd ~/Vircadia/sources
# Make changes to the sources
cd ~/vircadia-builder
./vircadia-builder --keep-source --tag=master --build=domain-server,ice-server,assignment-client
```

The last build with the `--keep-source` parameter has the builder build without refetching
the sources so the modifications made to the source tree are included in the binaries.

[vircadia-builder] creates run scripts in `~/Vircadia/install-master` and I run each
of the services with small scripts:

run-ice-server.sh:

```sh
#! /bin/bash
cd ~/Vircadia/install_master
export HIFI_METAVERSE_URL=http://192.168.86.56:9400
./run_ice-server --url ${HIFI_METAVERSE_URL}
```

run-domain-server.sh:

```sh
#! /bin/bash
cd ~/Vircadia/install_master

FORCETEMPNAME=--get-temp-name
export HIFI_METAVERSE_URL=http://192.168.86.56:9400
export ICE_SERVER=192.168.86.56:7337
./run_domain-server -i ${ICE_SERVER} ${FORCETEMPNAME}
```

The IP addresses above are for my development environment: 192.168.86.41 is
the Linux box running the metaverse-server and Interface, and
192.168.86.56 is my Linux box running the ice-server and domain-server.
Be sure to change these for your network setup.

The `--get-temp-name` parameter is required to start a domain-server
the first time
as it instructs the domain-server to fetch a temporary domain name
if it is not configured from a previous run.
The configuration of the
domain-server ends up in `.local/share/Vircadia/domain-server`.

Note that the ice-server and the domain-server get an environment variable
that points them at the new metaverse-server. This setting works for most of
the C++ code. The Javascript and QML code, on the other hand, need other
changes.

Once everything is built, the process is:

1. Start metaverse-server
1. Start ice-server
1. Start domain-server
1. Start Interface

## Modifications

As of 20200819 (August 19, 2020), the Vircadia sources have HighFidelity
URLs embedded in them. To run a domain-server and Interface that point to
another metaverse-server, the sources must be modified. The notes above
describe how to build for these changes.

The two known places that need modification are
`libraries/networking/src/NetworkingConstants.h`
and
`domain-server/resources/web/js/shared.js`.

There is a PR that incorporates these changes for the **BlueStuff** test
metaverse. This pull request has a build of the domain-server and
Interface with pointers to the test domain.
See [PR472](https://github.com/vircadia/vircadia/pull/472)
for built images.

[Vircadia]: https://github.com/vircadia/vircadia
[vircadia-builder]: https://github.com/vircadia/vircadia-builder
[Docker]: https://docker.io/
[DigitalOcean]: https://DigitalOcean.com/
[Running Docker Image]: ./RunningDockerImage.md
