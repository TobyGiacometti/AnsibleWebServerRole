# Ansible Web Server Role

An [Ansible][1] role that manages a web server.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
    - [Variables](#variables)
    - [HTTP Security Headers](#http-security-headers)
    - [Example](#example)

## Features

- Sets up an [Apache HTTP Server][2].
- Optimizes the web server configuration for increased security and performance.
- Manages the web server configuration for sites.
- Offers PHP support using PHP-FPM.
- Facilitates secure deployments.

## Requirements

- Debian GNU/Linux 10 (Buster) on managed host

## Installation

Use [Ansible Galaxy][3] to install `tobygiacometti.web_server`. Check out the Ansible Galaxy [content installation instructions][4] if you need help.

## Usage

To get general guidance on how to use Ansible roles, visit the [official documentation][5].

### Variables

- `web_server_weekly_reload`: Boolean indicating whether the web server configuration should be reloaded once a week. This is especially useful if TLS certificates are being managed automatically using a service like Let's Encrypt. The reload is executed gracefully to not impact any clients. Defaults to `true`.
- `web_server_deploy_ssh_keys`: SSH authorized keys (one per line) for the `www-deploy` user. The user has read/write access to web server document root directories and can therefore be used to run deployments through SSH. If not specified, SSH access for the user is disabled.
- `web_server_sites`: Sites that are served by the web server. This variable takes a list of dictionaries with following key-value pairs:
    - `main_domain`: Domain through which the site can be accessed. The provided domain is also used as an internal site ID. As a result, a unique main domain must be provided for each site.
    - `alt_domains`: List of additional domains through which the site can be accessed. Please note that wildcards (`*`) are supported.
    - `main_domain_redirect`: Boolean indicating whether requests using an alternative domain should be redirected to the main domain. Defaults to `true`.
    - `doc_root`: Relative path to the site's document root (absolute paths are not supported). The path is relative to `/var/www/<main-domain>`. If not specified, `/var/www/<main-domain>` is used as the document root. Please note that the document root is not automatically removed.
    - `tls_cert_files`: TLS certificate files that are used to enable HTTPS support for the site. Unencrypted requests are still possible but lead to a server-side/client-side HTTPS redirect. If not specified, HTTPS access is disabled. This variable takes a dictionary with following key-value pairs:
        - `cert`: Path to the TLS certificate.
        - `key`: Path to the TLS certificate private key.
    - `php`: Boolean indicating whether PHP should be enabled for the site. Defaults to `true`.
    - `far_future_expires`: Boolean indicating whether static assets should be served with a far-future `Expires` HTTP header. Defaults to `false`.
    - `extra_directives`: Additional [Apache HTTP server directives][6] for the site.

### HTTP Security Headers

The web server sends following HTTP security headers (with a strict value) by default:

- `Content-Security-Policy`
- `Referrer-Policy`
- `X-Content-Type-Options`
- `X-Frame-Options`
- `X-XSS-Protection`
- `Strict-Transport-Security`

Those headers can be overridden as follows:

- Use the `extra_directives` parameter in conjunction with the [Apache `Header` directive][7].
- Send headers from the web application.

At the very least, a custom `Content-Security-Policy` header will have to be sent to override the strict default.

### Example

```yaml
- hosts: web_servers
  roles:
    - role: tobygiacometti.web_server
      vars:
        web_server_deploy_ssh_keys: |
          ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILLvN9CeszM2AVXN71S6oykDvK/zmcS5M4v9fUAFLS7W
          ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE2t3yUgO8zLwut+2asWzhcebhrnsKwUF4KBeB8SyhRS
        web_server_sites:
          - main_domain: www.domain.example
            alt_domains:
              - domain.example
            doc_root: www
            tls_cert_files:
              cert: /etc/certs/www.domain.example/fullchain
              key: /etc/certs/www.domain.example/key
            extra_directives: |
              Header always set Content-Security-Policy "default-src 'self'; base-uri 'none'; form-action 'self'; frame-ancestors 'none'; block-all-mixed-content" "expr=%{CONTENT_TYPE} =~ m#text/html#i"
```

[1]: https://www.ansible.com
[2]: https://httpd.apache.org
[3]: https://galaxy.ansible.com
[4]: https://galaxy.ansible.com/docs/using/installing.html
[5]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html
[6]: https://httpd.apache.org/docs/current/mod/
[7]: https://httpd.apache.org/docs/current/mod/mod_headers.html#header
