# CoreOS Clair Lab

[CoreOS Clair](https://github.com/coreos/clair/) is a vulnerability analysis service.

This lab goals:

- Use [CentOS 7.x Vagrant Box](https://atlas.hashicorp.com/centos/boxes/7)
- Use [Docker Official Engine](https://docs.docker.com/engine/installation/linux/centos/)
- Use Go through the [`golang` container](https://hub.docker.com/_/golang/)

## Vagrant

This is a CentOS 7 lab, so go for it, with Virtualbox:

    $ vagrant up --provider=virtualbox

It will automatically provision with latest Docker from official repos.

## Launch Clair

Pull the Clair image:

    $ sudo docker pull quay.io/coreos/clair:latest

Change the SELinux context for the configuration inside the Vagrant shared folder:

    $ chcon -Rt svirt_sandbox_file_t ~/sync/config

Finally run and expose ports 6060 (API) and 6061 (public health checks), with the shared configuration:

    $ sudo docker run -p 6060:6060 -p 6061:6061 \
    -v `pwd`/sync/config:/config:ro \
    -v /tmp:/tmp -v /tmp:/db \
    quay.io/coreos/clair:latest \
    --config=/config/config.memstore.yaml

Wait for ~15mn for the CVEs to download and DB to populate.

## Local Analysis

Let's try to use [this simple tool](https://github.com/coreos/clair/contrib/analyze-local-images) built by CoreOS folks, using the _golang_ container to generate the binary:

    $ mkdir ~/go
    $ chcon -Rt svirt_sandbox_file_t go
    $ sudo docker run -it -v `pwd`/go:/go golang go get -u github.com/coreos/clair/contrib/analyze-local-images

You may want to update your `PATH`:

    $ PATH=$PATH:~/go/bin/

Grab a known vulnerable image:

    $ sudo docker pull ubuntu:precise-20151020

Analyze it and find 10 vulnerabilities marked as _"high"_:

```
$ analyze-local-images [--minimum-priority High] ubuntu:precise-20151020
Saving ubuntu:precise-20151020
Getting image's history
Analyzing 4 layers
- Analyzing c9e6ee76344204d9ebfeeca47c560e346b1a22c8ac539eb20a341da6203cfd0a
- Analyzing 3ccefe51c8b26100c11ad9a4ec714b22902e137dbbbcc34a5ed7812cdbf225a5
- Analyzing 6161467ba4601bc605aee8dbed7a51e50fa5fcf29754a924c3841cbee2410209
- Analyzing 05d6470b216cd0f9724132bc3a84a3ffd2f099f305db91bcc626f92b85fb3c54
Getting image's vulnerabilities
- # CVE-2015-8126
[...]
- # CVE-2015-8472
[...]
- # CVE-2013-7422
[...]
- # CVE-2015-7547
[...]
- # CVE-2015-8390
[...]
- # CVE-2016-1283
[...]
- # CVE-2015-8394
[...]
- # CVE-2015-2327
[...]
- # CVE-2015-8387
[...]
- # CVE-2015-0860
[...]
```

At the time of this writing (2016/2/24), the official _golang_ Docker image has 6 _High_ vulnerabilities and 2 _Critical_:

    $ ~/go/bin/analyze-local-images --minimum-priority High golang | grep "# CVE"
    - # CVE-2016-1283
    - # CVE-2012-0954
    - # CVE-2012-3587
    - # CVE-2015-2059
    - # CVE-2015-4844
    - # CVE-2016-0494
    - # CVE-2015-5600
    - # CVE-2013-7445

## API Examples

Get Engine an API versions:

    $ curl -s localhost:6060/v1/versions | python -m json.tool

Internal health check:

    $ curl -s localhost:6060/v1/health | python -m json.tool

Public API health check:

    $ curl -s localhost:6061 | python -m json.tool

Get information about a vulnerability:

    $ curl -s localhost:6060/v1/vulnerabilities/CVE-2015-0235 | python -m json.tool

## Sources

- [Run Clair](https://github.com/coreos/clair/blob/master/docs/Run.md)
- [API](https://github.com/coreos/clair/blob/master/docs/API.md)
- [Contrib](https://github.com/coreos/clair/tree/master/contrib)
