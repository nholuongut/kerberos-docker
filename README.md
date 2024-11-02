# README - Kerberos/Docker

![](https://i.imgur.com/waxVImv.png)
### [View all Roadmaps](https://github.com/nholuongut/all-roadmaps) &nbsp;&middot;&nbsp; [Best Practices](https://github.com/nholuongut/all-roadmaps/blob/main/public/best-practices/) &nbsp;&middot;&nbsp; [Questions](https://www.linkedin.com/in/nholuong/)
<br/>

Kerberos/Docker is a project to run easily a **MIT Kerberos V5** architecture in a cluster of **docker containers**. It is really useful for running integration tests of project using Kerberos or for learning and testing Kerberos solution and administration.

<p align="center">
  <img src="./doc/kerberos-docker-logo.png" width=200/>
</p>

See: [MIT Kerberos V5](https://web.mit.edu/kerberos/) and [Docker](https://www.docker.com/).

## Prerequisites

Use an **operating system compatible with docker**, and install:  
- **docker-ce** (without `sudo` for running docker command and with `overlay2` driver).  
- **docker-compose**
- **GNU Make** (if not already available).  
- **GNU Bash** (if not already available).  
- **Python 3** (if not already available, with `pip` and `virtualenv`).  
- **Java 8 and Maven 3** (if not already available).  

To check compatible version, see `./.ci/check-version.sh` traces on Travis CI web interface:  

https://github.com/nholuongut/kerberos-docker

To run tests, install **Bats**, see `./.ci/install.sh`.

Note:
- For Linux and MacOS workstation, it works on all distributions. 
- For Windows workstation, it works on [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about) 
with Ubuntu, but connect to docker container to interact with Kerberos server. 

## Usage

After installation, there are 3 docker containers with python web server on each one to check if it turns:

- `krb5-machine-example-com`
- `krb5-kdc-server-example-com`
- `krb5-service-example-com`

The goal is to connect from `krb5-machine-example-com` to `krb5-service-example-com` with ssh and Kerberos authentication (using GSSAPIAuthentication).

Here cluster architecture:

<p align="center">
  <img src="./doc/kerberos-docker-architecture.png" width=700/>
</p>


## Installation

Execute:

~~~
make install
~~~

See `Makefile` with `make usage` for all commands.

## Uninstallation

Execute:

~~~
make clean
~~~

To delete `network-analyser`, do `./network-analyser/clean-network-analyser.sh`.

For ubuntu operating system on docker container:

To delete `ubuntu:16.04` and `minimal-ubuntu:latest` docker images do `docker rmi ubuntu:16.04 minimal-ubuntu`.

## Possible improvements

* Add LDAP (or not) for Kerberos architecture
* Add other connector and service (postgresql, mongodb, nfs, hadoop) only OpenSSH for the moment
* Add Java, python or C using GSS API ... to connect with Kerberos authentication


## Test and Continous Integration

This project uses [Travis CI](https://www.travis-ci.org/) and
[Bash Automated Testing System (BATS)](https://github.com/bats-core/bats-core).

After installing `bast` (see version in Prerequisites part), you can test with `make test`.

## Network analyser

You can create a [wireshark](https://www.wireshark.org/) instance running in a docker container built from docker image named `network-analyser`.

See more details in `./network-analyser/README.md`.

## Debug and see traces

You can connect with interactive session to a docker container:

~~~
docker exec -it <container_name_or_id> bash
~~~

To debug kerberos (client or server):

~~~
export KRB5_TRACE=/dev/stdout
~~~

To debug ssh client:

~~~~
ssh -vvv username@host
~~~~

To debug ssh server:

~~~~
/usr/sbin/sshd -f /etc/ssh/sshd_config -d -e
~~~~

## Troubleshooting

**Kerberos services**

On `krb5-kdc-server-example-com` docker container, there are 2 Kerberos services `krb5-admin-service` and `krb5-kdc`:

~~~
supervisorctl status
~~~

See all opened ports on a machine:

~~~
netstat -tulpn
~~~


Check that each machine has a synchronized time (with `ntp` protocol and `date` to check).

See [Troubleshooting](https://web.mit.edu/kerberos/krb5-latest/doc/admin/troubleshoot.html) and
[Kerberos reserved ports](https://web.mit.edu/kerberos/krb5-1.5/krb5-1.5.4/doc/krb5-admin/Configuring-Your-Firewall-to-Work-With-Kerberos-V5.html).

**Conflict private IP addresses**

To create `example.com` network docker, the private sub-network `10.5.0.0/24`
should be free and private IP addresses `10.5.0.0/24` should free also. Check
your routage table with `route -n`, test free IP addresses with
`ping -c 1 -w 2 <host>`, and check request paths with `traceroute <host>`.

If the issue persists, you can do `make clean` or `docker network rm example.com`.

**Working on your computer (host machine) for debugging code**

Modify your `/etc/hosts` to resolve bidirectionally IP addresses with DNS of
the kerberos cluster:

~~~
# /etc/hosts
# ...

# Kerberos cluster
# IP FQDN hostname
10.5.0.1	krb5-machine-example-com.example.com krb5-machine-example-com
10.5.0.2	krb5-kdc-server-example-com.example.com krb5-kdc-server-example-com
10.5.0.3	krb5-service-example-com.example.com krb5-service-example-com

# ...
~~~

You can `ping krb5-kdc-server-example-com|10.5.0.2` Kerberos KDC server, and check if
Kerberos server port is opened: `nmap -A 10.5.0.2/32 -p 88` (or if SSH
server port : `nmap -A 10.5.0.3/32 -p 22`).

Now you can debug code and do `kinit bob` on host machine directly.

The order of `entries` and `names` is important in `/etc/hosts`.
To resolve name from IP address, the resolver takes the first one (horizontally) if multiple names
are possible; and to resolve IP address from name , the resolver takes the first entry (vertically)
if multiple IP addresses are possible: You can use `resolveip <IP|name>`, `getent hosts <IP|name>`
or just take a look at `/etc/hosts`.


## References

* ROBINSON Trevor (eztenia). **Kerberos**. Canonical Ltd. Ubuntu Article, November 2014. Link: https://help.ubuntu.com/community/Kerberos.
* MIGEON Jean. **Protocol, Installation and Single Sign On, The MIT Kerberos Admnistrator's how-to Guide**. MIT Kerberos Consortium, July 2008. p 62.
* BARRETT Daniel, SILVERMAN Richard, BYRNES Robert. **SSH, The Secure Shell: The Definitive Guide, 2nd Edition**. O'Reilly Media, June 2009. p. 672. Notes: Chapter 11. ISBN-10: 0596008953, ISBN-13: 978-0596008956
* GARMAN, Jason. **Kerberos: The Definitive Guide, 2nd Edition**. O'Reilly Media, March 2010. p. 272.  ISBN-10: 0596004036, ISBN-13: 978-0596004033.
* O’MALLEY Owen, ZHANG Kan, RADIA Sanjay, MARTI Ram, and HARRELL Christopher. **Hadoop Security Design**. Yahoo! Research Paper, October 2009. p 19.
*  MATTHIAS Karl, KANE Sean. **Docker: Up & Running**. O'Reilly Media, June 2015. p. 232. ISBN-10: 1491917571, ISBN-13: 978-1491917572.

# 🚀 I'm are always open to your feedback.  Please contact as bellow information:
### [Contact ]
* [Name: Nho Luong]
* [Skype](luongutnho_skype)
* [Github](https://github.com/nholuongut/)
* [Linkedin](https://www.linkedin.com/in/nholuong/)
* [Email Address](luongutnho@hotmail.com)
* [PayPal.me](https://www.paypal.com/paypalme/nholuongut)

![](https://i.imgur.com/waxVImv.png)
![](Donate.png)
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/nholuong)

# License
* Nho Luong (c). All Rights Reserved.🌟