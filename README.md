Puma rbenv Nginx Ansible Role
=============================

This Ansible role will ensure creation of app environment user,
install Ruby Puma with rbenv from official repositories, ensure Nginx virtual
host, manage apps with theirs UNIX sockets and create & setup SystemD init
scripts per instance.

You can launch this role as much as you need to, just ensure that `app_name`
or `app_puma_service_name` are different from one role call to another in order
to avoid collisions.


Requirements
------------

* Ansible >= 2.3
* One of these target systems:
    * Debian Jessie & Stretch
    * Ubuntu 16.04


Usage example
-------------

This is a simple example of how to make a Ruby webapp works.

For other languages/frameworks, or extra parameters, read [default variables](defaults/main.yml) for further details.
Normally, you will not be limited by the features offered.

By default, lots of behaviors are enabled such as Gzip or client static assets caching.

```yaml
- role: puma-rbenv-nginx
  app_name: cool-example.org
  app_puma_service_name: cool-example
  app_puma_exec_user: cool-example
  app_puma_ruby_version: 2.2.3
  app_puma_n_worker: 4
  app_puma_n_thread: 4
  app_nginx_ssl_enabled: true
  app_nginx_ssl_only: true
  app_nginx_ssl_cert_path: /etc/ssl/certs/example_ca-bundle.pem
  app_nginx_ssl_priv_path: /etc/ssl/priv/example.key
  app_nginx_static_cache_assets_enabled: false
  app_nginx_static_cache_assets_expire_hours: 10
  app_nginx_static_paths:
    - /favicon.ico
    - /stylesheets/
    - /javascripts/
    - /images/
    - /plugin_assets/
    - /help/
    - /themes/
  app_nginx_limitreq_enabled: true
  app_nginx_limitreq_per_second: 10 # max requests per ip per second
```

Simple as that.


How the userspaces works then
-----------------------------

By default, everything will get in `/opt`.

For instance, if your `app_puma_exec_user` is 'cool-example', then a user
will be created with `/opt/cool-example` as home directory.

You can change this behavior to replace `/opt` with another path by setting
the `app_puma_home_root` variable to the global base path that you want to.

In the `/opt/cool-example`, there a `src` directory that will get created.
That's where Puma will look for a `config.rh` to launch as expected.

For example, that's where you will put your Ruby on Rails application.

If you want to install gems in the userspace for example, 


How to enable rbenv outside puma-rbenv-nginx
--------------------------------------------

Ansible commands such as `gem` or `bundler` won't work. You will need
to use shell when you want to make operations in the rbenv.

There's an example how to run `bundler install` :

```yaml
- name: Ensure Bundler packages present
  shell: |
    . {{ app_rbenv_activation_script_path }} && cd src && bundle install && rbenv rehash
  args:
    executable: /bin/bash
```

Or how to install many gems :

```yaml
- name: Ensure some Gems present
  shell: |
    . {{ app_rbenv_activation_script_path }} && cd src && gem install {{ item }} && rbenv rehash
  args:
    executable: /bin/bash
  with_items:
    - rails
    - nokogiri
    - puma
```


Daemons / Services
------------------

Once you've runned the role, you can start and stop the app server this way :

```sh
service cool-example start
service cool-example stop
service cool-example restart
```


Conclusion
----------

We absolutely appreciate patches, feel free to contribute directly on the GitHub project.

Repositories / Development website / Bug Tracker:
- https://github.com/savoirfairelinux/ansible-puma-rbenv-nginx.git

Do not hesitate to join us and post comments, suggestions, questions and general feedback directly on the issues tracker.

**Website :** https://www.savoirfairelinux.com/
