dehydrated
==============

Install and configure `dehydrated`. Create user for privilege dropping
and cron configuration for certificate renewals.


**dehydrated is working with your private keys so be careful and review
the code of this [ansible role](https://github.com/martin-v/ansible-dehydrated)
an the used [dehydrated script](https://github.com/lukas2511/dehydrated/blob/116386486b3749e4c5e1b4da35904f30f8b2749b/dehydrated).**


For an example setup with nginx as https proxy take a look at ansible role [martin-v/ansible-nginx_https_only](https://github.com/martin-v/ansible-nginx_https_only)

Requirements
------------

The role installs on host:

  * openssl
  * curl
  * sed
  * grep
  * mktemp
  * git

This role need a webserver who serves the directory configured in `dehydrated_challengesdir`
(default: `/var/www/dehydrated/`) at location
`http://<your-domain>/.well-known/acme-challenge/` for all certificate
request domains.


Role Variables
--------------

### Required Variables:

#### dehydrated_contactemail

Address for the letsencrypt account. Mostly for certificate expiration notices,
but should be not happen if the cron job works fine.

    dehydrated_contactemail: certmaster@example.com


#### dehydrated_letsencrypt_agreed_terms

To accept the letsencrypt terms of service set the variable
`dehydrated_letsencrypt_agreed_terms` to the current license url.
You find the actual url at https://acme-v01.api.letsencrypt.org/terms.

    dehydrated_letsencrypt_agreed_terms: https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf


#### dehydrated_domains

List of domains for certificate requests. For each line a certificate will
be created, in folder `/etc/dehydrated/certs/` with the name of the first
domain in line. The first domain is the common name, the other in line will
be alternate names for the certificate.

    dehydrated_domains: |
      example.com
      example.org www.example.org blog.example.org


#### dehydrated_deploy_cert

The Certificates must be readable for services like apache or dovecot.
But only the specific services should be allowed to read the certificate
for this service. So we must change the owner/group to a specific value
for each certificate. For security reasons this can be only done by root
user.

To have a generic solution the variable `dehydrated_deploy_cert`
exists. This variable must contain bash script for certificate
deployments. Typical tasks on deployment are copy certificate to other
directories, change file owner/permissions and restart services.

This code is called similar as normal dehydrated hooks, but after the
complete dehydrated run and with root permissions. The code is called
once for each certificate that has been produced.

Parameters:

* `DOMAIN`
  The primary domain name, i.e. the certificate common name (CN).
* `KEYFILE` (Filename: privkey.pem)
  The path of the file containing the private key.
* `CERTFILE` (Filename: cert.pem)
  The path of the file containing the signed certificate.
* `FULLCHAINFILE` (Filename: fullchain.pem)
  The path of the file containing the full certificate chain.
* `CHAINFILE` (Filename: chain.pem)
  The path of the file containing the intermediate certificate(s).
* `TIMESTAMP` (Filename: chain.pem)
  Timestamp when the specified certificate was created.

Example:

    dehydrated_deploy_cert: |
      mkdir -p /etc/nginx/ssl/${DOMAIN}
      cp "${KEYFILE}" "${CERTFILE}" "${FULLCHAINFILE}" "${CHAINFILE}" /etc/nginx/ssl/${DOMAIN}
      chown root:root /etc/nginx/ssl/${DOMAIN}/*
      chmod 600 /etc/nginx/ssl/${DOMAIN}/*
      systemctl restart nginx.service


#### dehydrated_run_cron_on_every_ansible_run

This role trigger on each execution the cron script to create or update the certificates. To disabled this behavior
use:

    dehydrated_run_cron_on_every_ansible_run: false


### Optional Variables:

#### dehydrated_challengesdir

Directory for acme-challenge files. Your webserver should make this directory
public on location `http://<your-domain>/.well-known/acme-challenge/` for all domains listed
before. This directory will be created if it not exist. It should be only
writable for dehydrated user and readable by your webserver, this will
be enforced by this role.

    dehydrated_challengesdir: /var/www/dehydrated/


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
        - dehydrated_vars.yml
      roles:
        - martin-v.dehydrated


Example variables file
----------------------

    dehydrated_contactemail: certmaster@example.com

    dehydrated_letsencrypt_agreed_terms: https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf

    dehydrated_domains: |
      example.com
      example.org www.example.org blog.example.org

    dehydrated_deploy_cert: |
      mkdir -p /etc/nginx/ssl/${DOMAIN}
      cp "${KEYFILE}" "${CERTFILE}" "${FULLCHAINFILE}" "${CHAINFILE}" /etc/nginx/ssl/${DOMAIN}
      chown root:root /etc/nginx/ssl/${DOMAIN}/*
      chmod 600 /etc/nginx/ssl/${DOMAIN}/*
      systemctl restart nginx.service


Tips
----

To create certificates on ansible deployment, you can call the regular cron
script: `shell: "/etc/cron.weekly/dehydrated"`. The
[folder `tests`](https://github.com/martin-v/ansible-dehydrated/tree/master/tests)
contain a full running example.


For import from official letsencrypt client take a look at
[dehydrated import wiki page](https://github.com/lukas2511/dehydrated/wiki/Import-from-official-letsencrypt-client).


Open tasks
----------

[![Build Status travis](https://travis-ci.org/martin-v/ansible-dehydrated.svg?branch=master)](https://travis-ci.org/martin-v/ansible-dehydrated)
[![Build Status semaphore](https://semaphoreci.com/api/v1/martin-v/ansible-dehydrated/branches/master/badge.svg)](https://semaphoreci.com/martin-v/ansible-dehydrated)

0. Use [molecule](https://molecule.readthedocs.io/) for better tests
0. Reenable ansible-lint for etckeeper branch


License
-------

MIT

Author Information
------------------

This role was created in 2016 and improved in 2017 by [Martin V.](https://github.com/martin-v).
