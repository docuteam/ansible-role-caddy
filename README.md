# Ansible Role: Caddy

[![CI](https://github.com/simplificator/ansible-role-caddy/workflows/CI/badge.svg?event=push)](https://github.com/simplificator/ansible-role-caddy/actions?query=workflow%3ACI)

This role installs a plain caddy server on a Debian-based system and can, if you want, also place a basic reverse proxy configuration.

## Requirements

N/A

## Role Variables

If you only want to install Caddy, you don't need to set any variables. If you want to configure Caddy as a reverse proxy as well, you can provide an array of objects named `caddy_sites` with the following values:

* `additional_forwarding_ports`: Allows to define a list with additional ports where Caddy should listen for this domain and forward to HTTPS.
* `allowlist`: An array of IP addresses in CIDR-notation which are allowed to access this site (Optional). All other visitors receive a 404 error.
* `useragent_blocklist`: An array of User-Agents which are blocked to access this site (Optional), wildcard characters (*) need to be used for broader matching.
* `certificate_file`: You can set this variable if you want to provide the certificate by yourself (Optional). The certificate needs permissions `0640`, with root as Owner and Caddy as Group.
* `certificate_key`: You can set this variable if you want to provide the certificate by yourself (Optional).
* `domain`: The domain caddy should listen to.
* `default_response_code`: The code caddy will respond if the route is not defined. If not set, Caddy default behavior responds with code `200`.

Afterwards, you can define a list of `routes` composing of the following values:

* `path`: Path that should be matched. Let it empty for everything or e.g. `/api/*` for something specific.
* `ignore_allowlist`: If `true` the site's `allowlist` will not be applied to this route, thus making this route publicly available. Defaults to `false`.
* `reverse_proxy_destination`: Where the requested should be proxied.
* `strip_prefix`: If set, the matched `path` will be removed from the request to the destination system. This means, if somebody requests the route `/api/v1/hello` at the reverse proxy and you set `/api/*` as path, the request will be sent as `/v1/hello` to the destination system.

Additionally, you can also define a list of `redirects` composing of the following values:

* `source`: Path that should be matched. Let it empty for everything or e.g. `/api/*` for something specific.
* `target`: Target location. Becomes the response's Location header.
* `code`: HTTP status code to use for the redirect. Can be an integer in the 3xx range, or 401, `temporary` for temporary redirect (302), `permanent` for a permanent redirect (301, default), `html` to use an HTML document to perform the redirect (will redirect browsers, but not API clients).

Certificates, domain etc. are always defined for one site and cannot be redefined for a route.

The parameter `additional_template_path` defines a path to a template that gets appended to a Caddy site. This option can be helpful if you generally use the options from the YAML for your configuration but have one or two routes with special requirements. The template is inserted after rendering `allowlist` so you can apply rewrites to your routes before the code generated by this role is applied.

## Dependencies

None.

## Example Playbooks

Basic installation:

```yaml
---
- name: Converge
  hosts: all
  become: true

  roles:
    - role: simplificator.caddy
```

With reverse proxy configuration and redirects:

```yaml
---
- name: Converge
  hosts: all
  become: true

  roles:
    - role: simplificator.caddy

  vars:
    caddy_sites:
      - domain: example.com
        routes:
          - path: ''
            reverse_proxy_destination: 192.168.50.2
        redirects:
          - source: ''
            target: '/'
        allowlist:
          - 8.8.8.8/32
        additional_forwarding_ports:
          - '8080'
          - '1337'
```
