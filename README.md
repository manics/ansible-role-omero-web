OMERO Web
=========

Installs and configures OMERO.web and Nginx.
Uses a conf.d style configuration directory for managing the OMERO.web configuration.


Role Variables
--------------

All variables are optional, see `defaults/main.yml` for the full list

- `omero_web_release`: The version of OMERO.web to install, default `latest`
- `omero_web_upgrade`: Upgrade OMERO.web if the current version does not match `omero_web_release`
- `omero_web_always_reset_config`: Clear the existing configuration before regenerating, default `True`
- `omero_web_config_set`: A dictionary of `config-key: value` which will be used for the initial OMERO.web configuration, default empty.
  `value` can be a YAML object (e.g. string, list, dictionary) that will be automatically converted to quoted JSON when passed to OMERO.web.
- `omero_web_ice_version`: The ice version.
- `omero_web_systemd_setup`: Create and start the `omero-web` systemd service, default `True`


Configuring OMERO.web
---------------------

This role regenerates the OMERO.web configuration file using the configuration files and helper script in `/opt/omero/web/config`.
`omero_web_config_set` can be used for simple configurations, for anything more complex consider creating one or more configuration files under: `/opt/omero/web/config/` with the extension `.omero`.

Manual configuration changes (`omero config ...`) will be lost following a restart of `omero-web` with systemd, you can disable this by setting `omero_web_always_reset_config: False`.
Manual configuration changes will never be copied during an upgrade.

See https://github.com/openmicroscopy/design/issues/70 for a proposal to add support for a conf.d style directory directly into OMERO.


Example Playbooks
-----------------

OMERO.web with the default backend server, `localhost:4064`:

    - hosts: localhost
      roles:
        - role: openmicroscopy.omero-web

OMERO.web with a custom configuration using `omero_web_config_set`:

    - hosts: localhost
      roles:
        - role: openmicroscopy.omero-web
          omero_web_config_set:
            omero.web.server_list:
              - [omero.example.org, 4064, omero-example]
            omero.web.public.enabled: True
            omero.web.public.server_id: 1
            omero.web.public.user: public
            omero.web.public.password: secret-password

OMERO.web with a custom configuration using a configuration file `web-custom-config.omero`:

    - hosts: localhost
      roles:
        - role: openmicroscopy.omero-web
      tasks:
        - copy:
            content: >
              config set omero.web.server_list '[["omero.example.org", 4064, "omero-example"]'
            dest: /opt/omero/web/config/web-custom-config.omero
          notify:
            - restart omero-web


Author Information
------------------

ome-devel@lists.openmicroscopy.org.uk
