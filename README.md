# Ansible Role: HAProxy

![](https://github.com/kevincoakley/ansible-role-haproxy/workflows/Molecule%20Test/badge.svg)

Installs HAProxy on RedHat/CentOS and Debian/Ubuntu Linux servers.

**Note**: This role _officially_ supports HAProxy versions 1.4 or 1.5. Future versions may require some rework.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    haproxy_socket: /var/lib/haproxy/stats

The socket through which HAProxy can communicate (for admin purposes or statistics). To disable/remove this directive, set `haproxy_socket: ''` (an empty string).

    haproxy_chroot: /var/lib/haproxy

The jail directory where chroot() will be performed before dropping privileges. To disable/remove this directive, set `haproxy_chroot: ''` (an empty string). Only change this if you know what you're doing!

    haproxy_user: haproxy
    haproxy_group: haproxy

The user and group under which HAProxy should run. Only change this if you know what you're doing!

    haproxy_frontends: 
      - name: 'first_frontend'
        bind:
          - address: '*'
            port: 80
          - address: '*'
            port: 443
            ssl: true
            pem_local: '/etc/ansible/host_files/frist/first.pem'
            pem_proxy: '/etc/ssl/certs/first.pem'
        ssl_redirect: true
        mode: 'http'
        backend_name: 'first_backend'
      - name: 'second_frontend'
        bind:
          - address: '*'
            port: 2222
        mode: 'tcp'
        option:
          - 'tcplog clf'
        backend_name: 'second_backend'
    

HAProxy frontend configuration directives.

    haproxy_backends:
      - name: 'first_backend'
        mode: 'http'
        balance_method: 'roundrobin'
        option:
          - 'forwardfor'
          - 'httpchk HEAD / HTTP/1.1\r\nHost:localhost'
        http_auth_userlist: first_userlist
        cookie: true
        servers:
          - name: app1
            address: 192.168.0.1:8080
            options: 'cookie app1 check'
          - name: app2
            address: 192.168.0.2:8080
            options: 'cookie app2 check'
      - name: 'second_backend'
        mode: 'tcp'
        balance_method: 'static-rr'
        option:
          - 'tcplog'
        servers:
          - name: ssh1
            address: 192.168.0.3:22
            options: 'check port 22 inter 12000'

HAProxy backend configuration directives.

    userlists:
      - name: first_userlist
        users:
          - username: test
            password_type: insecure-password
            password: abc123

User List configuration for HTTP Basic Auth

## Dependencies

None.

## Example Playbook

    - hosts: balancer
      become: yes
      become_method: sudo
      roles:
        - { role: kevincoakley.haproxy }

## License

MIT / BSD
