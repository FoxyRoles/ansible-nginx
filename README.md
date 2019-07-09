# ansible-nginx

Setups nginx from [ondrej/nginx](https://launchpad.net/~ondrej/+archive/ubuntu/nginx) stable PPA with HTTP/2

## Example playboot

```yaml
---
- hosts: myserver
  user: root
  sudo: False
  roles:
   - sunfoxcz.nginx
     php_default_port: 9000
     nginx_uid: 1001
     nginx_gid: 1001
     nginx_user: web
     nginx_group: web
     nginx_user_shell: /bin/bash
     nginx_zones:
      - mydomain.tld
```

## License

Licensed under MIT license. See [LICENSE](LICENSE.md) for details.
