letsencrypt.sh
==============

Install and configure `letsencrypt.sh`. Create user for privilege dropping
and cron configuration for certificate renewals.


**letsencrypt.sh is working with your private keys so be careful and review
the code of this [ansible role](https://github.com/martin-v/ansible-letsencryptsh)
an the used [letsencrypt.sh script](https://github.com/lukas2511/letsencrypt.sh/blob/2099c77fee3e7a15c5cea93063248af4569bf8de/letsencrypt.sh).**


Requirements
------------

The role installs on host:

  * openssl
  * curl
  * sed
  * grep
  * mktemp
  * git

This role need a webserver who serves the directory configured in `lcsh_challengesdir`
(default: `/var/www/letsencrypt.sh/`) at location
`http://<your-domain>/.well-known/acme-challenge/` for all certificate
request domains.


Role Variables
--------------

### Required Variables:

#### lcsh_contactemail

Address for the letsencrypt account. Mostly for certificate expiration notices,
but should be not happen if the cron job works fine.

    lcsh_contactemail: certmaster@example.com


#### lcsh_domains

List of domains for certificate requests. For each line a certificate will
be created, in folder `/etc/letsencrypt.sh/certs/` with the name of the first
domain in line. The first domain is the common name, the other in line will
be alternate names for the certificate.

    lcsh_domains: |
      example.com
      example.org www.example.org blog.example.org


#### lcsh_deploy_cert

The Certificates must be readable for services like apache or dovecot.
But only the specific services should be allowed to read the certificate
for this service. So we must change the owner/group to a specific value
for each certificate. For security reasons this can be only done by root
user.

To have a generic solution the variable `lcsh_deploy_cert`
exists. This variable must contain bash script for certificate
deployments. Typical tasks on deployment are copy certificate to other
directories, change file owner/permissions and restart services.

This code is called similar as normal letsencrypt.sh hooks, but after the
complete letsencrypt.sh run and with root permissions. The code is called
once for each certificate that has been produced.

Parameters:

* `DOMAIN`
  The primary domain name, i.e. the certificate common name (CN).
* `KEYFILE`
  The path of the file containing the private key.
* `CERTFILE`
  The path of the file containing the signed certificate.
* `FULLCHAINFILE`
  The path of the file containing the full certificate chain.
* `CHAINFILE`
  The path of the file containing the intermediate certificate(s).

Example:

    lcsh_deploy_cert: |
      mkdir -p /etc/nginx/ssl/${DOMAIN}
      cp "${KEYFILE}" "${CERTFILE}" "${FULLCHAINFILE}" "${CHAINFILE}" /etc/nginx/ssl/${DOMAIN}
      chown root:root /etc/nginx/ssl/${DOMAIN}/*
      chmod 600 /etc/nginx/ssl/${DOMAIN}/*
      systemctl restart nginx.service


### Optional Variables:

#### lcsh_challengesdir

Directory for acme-challenge files. Your webserver should make this directory
public on location `http://<your-domain>/.well-known/acme-challenge/` for all domains listed
before. This directory will be created if it not exist. It should be only
writable for letsencrypt.sh user and readable by your webserver, this will
be enforced by this role.

    lcsh_challengesdir: /var/www/letsencrypt.sh/


#### More variables

There are also some unusual variables for super user who need more control,
for details take look at `defaults/main.yml`


Dependencies
------------

None.


Example Playbook
----------------

    - hosts: all
      remote_user: root
      vars_files:
        - letsencryptsh_vars.yml
      roles:
        - martin-v.letsencryptsh



Example variables file
----------------------

    ---

    lcsh_contactemail: certmaster@example.com

    lcsh_domains: |
      example.com
      example.org www.example.org blog.example.org

    lcsh_deploy_cert: |
      mkdir -p /etc/nginx/ssl/${DOMAIN}
      cp "${KEYFILE}" "${CERTFILE}" "${FULLCHAINFILE}" "${CHAINFILE}" /etc/nginx/ssl/${DOMAIN}
      chown root:root /etc/nginx/ssl/${DOMAIN}/*
      chmod 600 /etc/nginx/ssl/${DOMAIN}/*
      systemctl restart nginx.service


Tips
----

To create certificates on ansible deployment, you can call the regular cron
script: `shell: "/etc/cron.weekly/letsencrypt.sh"`. The
[folder `tests`](https://github.com/martin-v/ansible-letsencryptsh/tree/master/tests)
contain a full running example.


For import from official letsencrypt client take a look at
[letsencrypt.sh import wiki page](https://github.com/lukas2511/letsencrypt.sh/wiki/Import-from-official-letsencrypt-client).


Open tasks
----------

[![Build Status travis](https://travis-ci.org/martin-v/ansible-letsencryptsh.svg?branch=master)](https://travis-ci.org/martin-v/ansible-letsencryptsh)
[![Build Status semaphore](https://semaphoreci.com/api/v1/martin-v/ansible-letsencryptsh/branches/master/badge.svg)](https://semaphoreci.com/martin-v/ansible-letsencryptsh)

0. Write <del>more</del> tests
0. Example documentation for nginx and apache configuration


License
-------

MIT

Author Information
------------------

This role was created in 2016 by [Martin V](https://github.com/martin-v).
