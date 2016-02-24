# CoreOS Clair Lab

[CoreOS Clair](https://github.com/coreos/clair/) is a vulnerability analysis service.

## Vagrant

This is a CentOS 7 lab, so go for it, with Virtualbox:

    $ vagrant up --provider=virtualbox

## Install Docker

Automatically install Docker from official repositories:

    $ curl -fsSL https://get.docker.com/ | sh

Add the _vagrant_ user to the _docker_ group:

    $ sudo usermod -aG docker vagrant

Logout/Login

    $ exit

Enable and activate Docker service:

    $ sudo systemctl enable docker
    $ sudo systemctl start docker

Is the version OK ?

    $ docker --version
    Docker version 1.10.2, build c3959b1

## Launch Clair

Pull the Clair image:

    $ sudo docker pull quay.io/coreos/clair:latest

Change the SELinux context for the configuration inside the Vagrant shared folder:

    $ chcon -Rt svirt_sandbox_file_t ~/sync/config

Finally run and expose ports 6060 (API) and 6061 (public health checks), with the shared configuration:

    $ sudo docker run -p 6060:6060 -p 6061:6061 -v `pwd`/sync/config:/config:ro quay.io/coreos/clair:latest --config=/config/config.memstore.yaml

Wait for ~15mn for the CVEs to download and DB to populate.

## API Examples

Get Engine an API versions:

    $ curl -s localhost:6060/v1/versions | python -m json.tool

Internal health check:

    $ curl -s localhost:6060/v1/health | python -m json.tool

Public API health check:

    $ curl -s localhost:6061 | python -m json.tool

Get information about a vulnerability:

    $ curl -s localhost:6060/v1/vulnerabilities/CVE-2015-0235 | python -m json.tool

## Local Analysis

    $ mkdir ~/go
    $ chcon -Rt svirt_sandbox_file_t go
    $ sudo docker run -it -v `pwd`/go:/go golang go get -u github.com/coreos/clair/contrib/analyze-local-images
    $ PATH=$PATH:~/go/bin/

Grab a known vulnerable image:

    $ sudo docker pull ubuntu:precise-20151020
    $ analyze-local-images -endpoint "http://192.168.99.100:6060" -my-address "192.168.99.1" b1e4aeb0480d

## Sources

- [Run Clair](https://github.com/coreos/clair/blob/master/docs/Run.md)
- [API](https://github.com/coreos/clair/blob/master/docs/API.md)
- [Contrib](https://github.com/coreos/clair/tree/master/contrib)
