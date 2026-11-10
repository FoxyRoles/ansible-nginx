# ansible-nginx

An Ansible role that installs and configures nginx from the
[ondrej/nginx stable PPA](https://launchpad.net/~ondrej/+archive/ubuntu/nginx).

## Requirements

* Ansible Core 2.15 or newer, with fact gathering enabled.
* An Ubuntu host supported by the PPA, with APT and system groups `root` and `adm`.
* Root privileges, for example through `become: true`.
* The `openssl` command on the target host to generate Diffie–Hellman parameters.
* An available UID and GID for the web account, or values matching an existing
  account. The defaults are both `1000`, which may already belong to a login user.

PHP-FPM is optional and must be installed and configured separately if PHP or the
`/php-status` endpoint is needed. The role expects it on
`127.0.0.1:{{ php_default_port }}`.

## Example playbook

```yaml
---
- hosts: myserver
  become: true
  roles:
    - role: sunfoxcz.nginx
      nginx_zones:
        - example.com
        - example.org
```

## Role variables

No variables are mandatory. Defaults from `defaults/main.yml`:

```yaml
nginx_uid: 1000
nginx_gid: 1000
nginx_user: web
nginx_group: web
nginx_user_shell: /bin/bash

php_default_port: 9000
```

| Variable | Purpose |
| --- | --- |
| `nginx_uid`, `nginx_gid` | UID and GID of the web account and group. |
| `nginx_user`, `nginx_group` | Web account and group; nginx workers run as `nginx_user`. |
| `nginx_user_shell` | Login shell of the web account. |
| `php_default_port` | Local FastCGI port used by the `php` upstream and `/php-status`. |
| `nginx_zones` | Optional list of domain names. Omit it or use `[]` to configure only the default site. |

## Configuration

The role installs `ca-certificates` and `python3-debian`, then configures the PPA
with `ansible.builtin.deb822_repository` in `/etc/apt/sources.list.d/nginx.sources`.
The signing key is downloaded from Ubuntu's keyserver using the full fingerprint
published on the PPA page and scoped to this repository through `Signed-By`.
During migration, the role removes its former `/etc/apt/sources.list.d/nginx.list`
file after creating the new source. It refreshes the APT cache before installing
nginx.

The role enables and starts nginx, installs `libnginx-mod-http-brotli-filter`,
and configures dynamic Brotli compression at level 6 alongside Gzip. It does not
install the Brotli static module or configure serving precompressed `.br` files.
Dynamic modules are loaded from `/etc/nginx/modules-enabled/*.conf`.

The role manages `/etc/nginx/nginx.conf`, compression and PHP upstream
configuration in `/etc/nginx/conf.d`, FastCGI parameters and limits, and a
`secure_files` snippet that blocks access to version-control metadata and selected
configuration files. Configuration changes notify a handler that restarts nginx.

### Sites and PHP

Site configurations are written to `/etc/nginx/sites.d`. The generated nginx
configuration includes this directory instead of `sites-enabled`; existing
`sites-available` and `sites-enabled` directories are left in place.

For each entry in `nginx_zones`, the role creates:

* `/etc/nginx/sites.d/<domain>`, serving `<domain>` and `*.<domain>` over HTTP on
  port 80, with IPv4 and IPv6 listeners.
* `/home/<nginx_user>/<domain>/www`, owned by the configured web account and group.
* Access and error log configuration pointing to
  `/var/log/nginx/<domain>.access.log` and `/var/log/nginx/<domain>.error.log`.

Sites use `index.html` or `index.php`, fall back to `/index.php` for missing paths,
and forward PHP requests to the local FastCGI upstream. Removing a domain from
`nginx_zones` does not delete its existing configuration or content directory.

The default site serves `/var/www/html` with an empty index page and provides
`/nginx-status` and `/php-status`, accessible only from `127.0.0.1`.

### TLS

The role creates `/etc/nginx/ssl` with mode `0700`, generates a 4096-bit
Diffie–Hellman parameter file with mode `0600`, and installs shared TLS settings
for TLS 1.2 and TLS 1.3. Parameter generation runs only when the file is absent and
can take some time on the first run.

Certificates and HTTPS virtual hosts must be configured separately; the generated
sites listen only on HTTP port 80.

### Logging

The role recursively sets ownership of `/var/log/nginx` to `root:adm`, grants the
owner read/write access and the group read access, preserves existing executable
bits, and grants directory traversal to the owner and group. Other users receive
no access (`u=rwX,g=rX,o=-`).

It also updates `/etc/logrotate.d/nginx` to create rotated logs with mode `0640`
and ownership `root:adm`.

## License

Licensed under the MIT license. See [LICENSE](LICENSE.md) for details.
