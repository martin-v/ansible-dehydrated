Role Name
=========

Install and configure
[`letsencrypt.sh`](https://github.com/lukas2511/letsencrypt.sh).
Create user for privilege dropping and cron configuration for certificate
renewals.


**letsencrypt.sh is working with your private keys so be careful and review
the code of this [ansible role](https://github.com/martin-v/ansible-letsencryptsh)
an the used [letsencrypt.sh script](https://github.com/lukas2511/letsencrypt.sh/blob/2099c77fee3e7a15c5cea93063248af4569bf8de/letsencrypt.sh).**


Requirements
------------

Installs on host:
  - openssl
  - curl
  - sed
  - grep
  - mktemp
  - git

This role need a webserver who serves the directory configured in `lcsh_challengesdir`
(default: `/var/www/letsencrypt.sh/`) at location
`http://<your-domain>/.well-known/acme-challenge/` for all certificate
request domains.


Role Variables
--------------

### Required Variables:

Address for the letsencrypt account. Mostly for certificate expiration notices,
but should be not happen if the cron job works fine.

    lcsh_contactemail: certmaster@example.com

List of domains for certificate requests. For each line a certificate will
be created, in folder `/etc/letsencrypt.sh/certs/` with the name of the first
domain in line. The first domain is the common name, the other in line will
be alternate names for the certificate.

    domanins: |
      example.com
      example.org www.example.org blog.example.org


### Optional Variables:

Directory for acme-challenge files. Your webserver should make this directory
public on location `http://<your-domain>/.well-known/acme-challenge/` for all domains listed
before. This directory will be created if it not exist. It should be only
writable for letsencrypt.sh user and readable by your webserver, this will
be enforced by this role.

    lcsh_challengesdir: /var/www/letsencrypt.sh/


There are also some more advanced variables for super user who need more control,
for details look at `defaults/main.yml`


Dependencies
------------

None.


Example Playbook
----------------

    - hosts: servers
      roles:
        - { role: martin-v.letsencryptsh }
      vars_files:
        - group_vars/letsencryptsh.yml

Example Variables file
----------------------

    ---

    lcsh_contactemail: certmaster@example.com

    domanins: |
      example.com
      example.org www.example.org blog.example.org


Open tasks
----------

[![Build Status](https://travis-ci.org/martin-v/ansible-letsencryptsh.svg?branch=master)](https://travis-ci.org/martin-v/ansible-letsencryptsh)

0. Umask for letsencryptsh user to paranoid mode
0. Call initial certificate requests
0. Write <del>more</del> tests
0. Example documentation for nginx and apache configuration
0. Example documentation for import from official letsencrypt client
0. Replace cron with systemd timer
0. Support hooks


License
-------

MIT

Author Information
------------------

This role was created in 2016 by [Martin V](https://github.com/martin-v).
