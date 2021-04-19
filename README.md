Ansible Role: SSL Certificates
==============================

An Ansible role that installs SSL certificates.

Table of Contents
-----------------

<!-- toc -->

- [Requirements](#requirements)
- [Role Variables](#role-variables)
  * [Re-Use Destination Variables](#re-use-destination-variables)
- [Dependencies](#dependencies)
- [Example Playbook](#example-playbook)
  * [Top-Level Playbook](#top-level-playbook)
  * [Role Dependency](#role-dependency)
- [License](#license)
- [Author Information](#author-information)

<!-- tocstop -->

Requirements
------------

- Ansible 2.4+

Role Variables
--------------

This role only needs one variable, although this one looks a bit verbose at
first. The variable is basically a list of certificates. Each certificate
requires a `key` and a `cert`, and optionally a `chain`.

```yml
ssl_certificates:
  - name: somehosts ssl certificate for blah.example.com
    key:
      content: '{{ vault_ssl_certificate_key }}'
      dest: /path/to/key.pem
    cert:
      src: path/to/cert.pem
      dest: /path/to/cert.pem
    chain:
      src: path/to/chain
      dest: /path/to/chain
  - name: somehosts ssl certificate for bippy.example.com
    key:
      ...
```

**Note:** It is recommended to [put the key in a vault][vault]!

**Note:** Ensure that Ansible can find the `src` files. `group_vars/group` and
`host_vars/host` are not automatically searched. If you want to keep the files
there, use e.g. `host_vars/host/example.com.pem`.

**Note:** `key`, `cert` and `chain` also allow to set a custom `setype`,
default is `cert_t`.

**Note:** `key` also allows a list to give additional read permissions via ACL
entries. This is for services that need access to the key which do not start as
**root** and then drop privileges.

```yml
ssl_certificates:
  - name: somehosts ssl certificate for blah.example.com
    key:
      content: '{{ vault_ssl_certificate_key }}'
      dest: /path/to/key.pem
      acl_users:
        - service-user-a
        - service-user-b
    cert:
      ...
```

### Re-Use Destination Variables

You can re-use the **destination** variables for the configuration variables of
other roles, e.g.:

```yml
---

ssl_certificates:
  - name: somehosts ssl certificate for blah.example.com
    ...
  - name: ...
    ...

# because ssl_certificates is a list, you need to index with [n]

apache_ssl_cert_key_file: '{{ ssl_certificates[0].key.dest }}'
apache_ssl_cert_file: '{{ ssl_certificates[0].cert.dest }}'
apache_ssl_cert_chain_file: '{{ ssl_certificates[0].chain.dest }}'

# here, the second key is used for postfix

postfix_smtp_tls_key_file: '{{ ssl_certificates[1].key.dest }}'
postfix_smtp_tls_cert_file: '{{ ssl_certificates[1].cert.dest }}'

postfix_smtpd_tls_key_file: '{{ ssl_certificates[1].key.dest }}'
postfix_smtpd_tls_cert_file: '{{ ssl_certificates[1].cert.dest }}'

...
```

Dependencies
------------

None.

Example Playbook
----------------

Add to `requirements.yml`:

```yml
---

- src: idiv-biodiversity.ssl_certificates

...
```

Download:

```console
$ ansible-galaxy install -r requirements.yml
```

### Top-Level Playbook

Write a top-level playbook:

```yml
---

- name: head server
  hosts: head

  roles:
    - role: idiv-biodiversity.ssl_certificates
      tags:
        - certificates
        - ssl-certificates

...
```

### Role Dependency

Define the role dependency in `meta/main.yml`:

```yml
---

dependencies:

  - role: idiv-biodiversity.ssl_certificates
    tags:
      - certificates
      - ssl-certificates

...
```

License
-------

MIT

Author Information
------------------

This role was created in 2019 by [Christian Krause][author] aka [wookietreiber
at GitHub][wookietreiber] and Dirk Sarpe aka [dirks at GitHub][dirks], both
systems administrators at the [German Centre for Integrative Biodiversity
Research (iDiv)][idiv], based on a draft by Ben Langenberg aka [sloan87 at
GitHub][sloan87].


[author]: https://www.idiv.de/en/groups_and_people/employees/details/61.html
[idiv]: https://www.idiv.de/
[dirks]: https://github.com/dirks
[sloan87]: https://github.com/sloan87
[wookietreiber]: https://github.com/wookietreiber
[vault]: https://docs.ansible.com/ansible/latest/user_guide/vault.html
