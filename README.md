Varnish
=======

An Ansible Role that installs Varnish on RedHat/CentOS or Debian/Ubuntu Linux.

Requirements
------------

Requires the EPEL repository on RedHat/CentOS (you can install it using the `geerlingguy.repo-epel` role).

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    varnish_version: "4.0"

Varnish version that should be installed. See `https://repo.varnish-cache.org/redhat/` for a listing of available versions (e.g. `3.0`, `4.0`, `4.1`). _Note: Ubuntu 16.04 "Xenial"_

    varnish_config_path: /etc/varnish

The path in which Varnish configuration files will be stored.

    varnish_default_vcl_template_path: default.vcl.j2

The default VCL file to be copied. Defaults the simple template inside `templates/default.vcl.j2`. This path should be relative to the directory from which you run your playbook.

    varnish_listen_port: "80"

The port on which Varnish will listen (typically port 80).

    varnish_admin_listen_host: "127.0.0.1"
    varnish_admin_listen_port: "6082"

The host and port through which Varnish will accept admin requests (like purge and status requests).

    varnish_storage: "file,/var/lib/varnish/varnish_storage.bin,256M"

How Varnish stores cache entries (this is passed in as the argument for `-s`). If you want to use in-memory storage, change to something like `malloc,256M`. Please read Varnish's [Getting Started guide](https://www.varnish-software.com/static/book/Getting_started.html) for more information.

    varnish_secret: "{{ inventory_hostname | to_uuid }}"

The secret/key to be used for connecting to Varnish's admin backend (for purge requests, etc.).

    varnish_backends_defaults:
      max_connections: 300
      first_byte_timeout: 300
      connect_timeout: 5
      between_bytes_timeout: 2
      probe_interval: 5
      probe_timeout: 1
      probe_window: 5
      probe_threshold: 3

This properties set the default values for the backends of the list that does not override them explicitly.

    varnish_backends:
      - name: localhost
        host: 127.0.0.1
        port: 8080
        max_connections: 300 (default)
        first_byte_timeout: 300 (default)
        connect_timeout: 5 (default)
        between_bytes_timeout: 2 (default)
        probe_url: /
        probe_request: | (default)
          "HEAD / HTTP/1.1"
          "Host: 127.0.0.1"
          "Connection: close"
          "User-Agent: Varnish Health Probe";
        probe_interval: 5 (default)
        probe_timeout: 1 (default)
        probe_window: 5 (default)
        probe_threshold: 3 (default)
        backend_for:
          - site-1.com

Here you can list your backends. They could be Apache or Nginx (or some other HTTP server) running on the same host or some other host (in which case, you might use port 80 instead). Each backend must have a name, host and port properties.

You may also set some extra properties to adjust the checks for the backends health. You can read more about this values (and other VCL configurations) in [this](https://info.varnish-software.com/blog/backends-load-balancing) and [this](https://info.varnish-software.com/blog/backends-load-balancing-part-2) posts.

The `|` denotes a multiline scalar block in YAML, so newlines are preserved in the resulting configuration file output.

If you would not use `directors`, you can use the `backend_for` property to define which sites are served by the backend.

    varnish_directors: []
      - name: default
        type: round_robin (default)
        backends:
          - name: localhost
            weight: 1 (only for hash and random directors)
        director_for:
          - site-1.com

If you would use `directors`, you should list them in this variable. Each director must have a name and a list of backends. The default `type` is `round_robin`, but you can use any of [the types supported by Varnish](https://www.varnish-cache.org/docs/trunk/reference/vmod_directors.generated.html).

Each backend in the list must have a name property, that should match the name of one of the previously defined backends. If you use `hash` or `random` director types, you can also define a `weight` for that backend.

You can use the `director_for` property to define which sites are served by the director.

If no director (or backend, in the case you don't use directors) match the http host of the request, Varnish will use the director (or backend) named `default`. If there is no director (or backend) named `default`, the first director (or backend) will be used.

    varnish_acl_purge_hosts:
      - localhost
      - 127.0.0.1
      - ::1

The IP addresses or hostnames of the hosts that can purge the Varnish Cache using the `PURGE` HTTP method.

    varnish_acl_editors_hosts:
      - localhost
      - 127.0.0.1
      - ::1

The IP addresses or hostnames of the hosts that can purge the Varnish Cache using a `no-cache` value in the `Cache-Control` header.

    varnish_vcl_recv_backend_useless_parameters: []

This property defines a list of parameters that would be remove from the requested URL before forwarding the request to the backend.

    varnish_vcl_recv_remove_cookies: []

This property defines a list of cookies that would be remove from the request before forwarding it to the backend.

    varnish_vcl_recv_unset_all_cookies_exceptions: []
      - host: site-1.com
        urls:
          - ^/user
      - url: ^/admin
        hosts:
          - site-1.com

This property list the conditions that should match a request so its Cookies would not be unset, disallowing the request to be cached. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

    varnish_vcl_recv_unset_all_cookies_conditions: []
      - host: site-1.com
        urls:
          - ^/static-page
      - url: ^/static-section
        hosts:
          - site-1.com

This property list the conditions that should match a request to unset all its Cookies, allowing the request to be cached. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

    varnish_vcl_recv_unset_all_cookies: False

When this property is set to `True` all the Cookies will we removed from the request to the backend. This allows to cache every request.

    varnish_vcl_recv_pass_exceptions: []
      - host: site-1.com
        urls:
          - ^/user
      - url: ^/admin
        hosts:
          - site-1.com

This property list the conditions that should match a request so it would not be passed to the backend, allowing the request to be cached. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

    varnish_vcl_recv_pass_conditions: []
      - host: site-1.com
        urls:
          - ^/static-page
      - url: ^/static-section
        hosts:
          - site-1.com

This property list the conditions that should match a request so it would be passed to the backend, disallowing the request to be cached. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

    varnish_vcl_recv_pass: False

When this property is set to `True` all the request will be passed to the backend. This disallows to cache every request.

    varnish_vcl_recv_pipe_exceptions: []
      - host: site-1.com
        urls:
          - ^/user
      - url: ^/admin
        hosts:
          - site-1.com

This property list the conditions that should match a request so it would not be piped to the backend, allowing the request to be cached. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

    varnish_vcl_recv_pipe_conditions: []
      - host: site-1.com
        urls:
          - ^/static-page
      - url: ^/static-section
        hosts:
          - site-1.com

This property list the conditions that should match a request so it would be piped to the backend, disallowing the request to be cached. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

    varnish_vcl_recv_pipe: False

When this property is set to `True` all the request will be piped to the backend. This disallows to cache every request.

    varnish_vcl_pipe_connection_close: True

Close the connection after piping the content to the client. Enabling this option would force the X-Forwarded-For header in all request to the backends.

    varnish_vcl_backend_response_unset_all_cookies_exceptions: []
      - host: site-1.com
        urls:
          - ^/user
      - url: ^/admin
        hosts:
          - site-1.com

This property list the conditions that should match a request made to a backend so the Set-Cookie headers of the response would not be unset, disallowing the request to be cached. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

The conditions of this properties must be at least the same defined for `varnish_vcl_recv_unset_all_cookies_exceptions`.

    varnish_vcl_backend_response_unset_all_cookies_conditions: []
      - host: site-1.com
        urls:
          - ^/static-page
      - url: ^/static-section
        hosts:
          - site-1.com

This property list the conditions that should match a request made to a backend so the Set-Cookie headers of the response would be unset, allowing the request to be cached. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

The conditions of this properties must be at least the same defined for `varnish_vcl_recv_unset_all_cookies_conditions`.

    varnish_vcl_backend_response_unset_all_cookies: False

When this property is set to `True` all the Set-Cookies headers will we removed from the response of the backend before forwarding it to the client. This allows to cache every request.

    varnish_vcl_backend_response_beresp_ttl: 0

Time to live in seconds for the response of the backend. At least one of the next three properties should be defined to use this value.

    varnish_vcl_backend_response_force_beresp_ttl_exceptions: []
      - host: site-1.com
        urls:
          - ^/user
      - url: ^/admin
        hosts:
          - site-1.com

This property list the conditions that should match a request made to a backend so the TTL of the backend response would not be overriden with `varnish_vcl_backend_response_beresp_ttl`. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

    varnish_vcl_backend_response_force_beresp_ttl_conditions: []
      - host: site-1.com
        urls:
          - ^/static-page
      - url: ^/static-section
        hosts:
          - site-1.com

This property list the conditions that should match a request made to a backend so the TTL of the backend response would be overriden with `varnish_vcl_backend_response_beresp_ttl`. The conditions may match all the URLs of a `host`, a `url` in all the hosts or filter by `urls` or `hosts` respectively.

    varnish_vcl_backend_response_force_beresp_ttl: False

When this property is set to `True` the TTL of all the backend responses would be overriden with `varnish_vcl_backend_response_beresp_ttl`.

    varnish_vcl_backend_response_beresp_grace: 6

Adds a grace period to the cached objects in case there is no healthy backend to serve the request.

    varnish_vcl_backend_error_response: "{{ lookup('file', 'files/503.j2') }}"

HTML code for the 503 (Backend unavailable) page.

    varnish_vcl_deliver_remove_headers: []

List of HTTP headers to remove from the backend response before forwarding it to the client.

Dependencies
------------

None.

Example Playbook
----------------

    - hosts: cacheservers
      vars_files:
        - vars/main.yml
      roles:
        - gcoop-libre.varnish

*Inside `vars/main.yml`*:

    varnish_secret: "[secret generated by uuidgen]"
    varnish_backends:
      - host: 127.0.0.1
        port: 8080
    ... etc ...

License
-------

GPLv2

Author Information
------------------

This role was created in 2016 by [gcoop Cooperativa de Software Libre](http://gcoop.coop).

It extends the functionality of the Varnish role created in 2014 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
