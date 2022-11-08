# ansible-nginx

Setups nginx from [ondrej/nginx](https://launchpad.net/~ondrej/+archive/ubuntu/nginx) stable PPA

## Why?

 * compiled against latest OpenSSL for HTTP/2 and TLS 1.3 support

## Example playbook

```yaml
---
- hosts: myserver
  user: root
  sudo: False
  roles:
   - sunfoxcz.nginx
     nginx_zones:
      - mydomain.tld
```

## Mandatory variables

None

## Optional variables

 * nginx_zones
 * nginx_package
 * nginx_uid
 * nginx_gid
 * nginx_user
 * nginx_group
 * nginx_user_shell
 * php_default_port

## Default values
```yaml
nginx_package: nginx-light # nginx-full is other option

nginx_uid: 1001
nginx_gid: 1001
nginx_user: web
nginx_group: web
nginx_user_shell: /bin/bash

php_default_port: 9000
```

## License

Licensed under MIT license. See [LICENSE](LICENSE.md) for details.
