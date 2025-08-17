Ansible Role: NUT
=================
Installs and configures [NUT](http://networkupstools.org/) (Nework UPS
tools) on Debian based systems.

Role Variables
--------------

Available variables are listed below, along with default values (see
`defaults/main.yml`):

### Generic Configuration

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `nut_managed_config` | `true` | If this is set to false, none of the following options will have any effect, that is any and all changes under `/etc/nut/` will be your responsibility. This can be desirable if you have complex configurations. |
| `nut_enable_service` | `true` | Whether to start the NUT service after configuration |
| `nut_mode` | `standalone` | NUT mode setting (see `man 5 nut.conf` MODE directive) |
| `nut_services` | `[nut-server.service, nut-monitor.service, nut.target]` | List of NUT services, excluding driver-related ones, to enable |
| `nut_users` | See example below | List of users for NUT configuration (replaces legacy user variables). See below for detailed schema. |
| `nut_ups` | See example below | List of UPS configurations with name, driver, device, description |
| `nut_ups_extra` | `maxretry = 3` | Additional configuration options directly placed in `ups.conf` |
| `nut_upsd_extra` | Multi-line config | Additional upsd daemon configuration directly placed in `upsd.conf` |
| `nut_maxretry` | `3` | **DEPRECATED** - Use `nut_ups_extra` instead |

### UPSMON Configuration

This settings are primarily used for the **local** `upsmon.conf` configuration.

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `nut_host` | `localhost` | Hostname of the NUT server to monitor |
| `nut_powervalue` | `1` | Power value for MONITOR directive (see `man 5 upsmon.conf`) |
| `nut_user` | `undefined` | **DEPRECATED** - Legacy user configuration, migrate to `nut_users` |
| `nut_password` | `undefined` | **DEPRECATED** - Legacy password configuration, migrate to `nut_users` |
| `nut_role` | `undefined` | **DEPRECATED** - Legacy role configuration, migrate to `nut_users` |
| `nut_upsmon_user` | Auto-derived | Use this variable to override how the upsmon **username** is derived. If you override, make sure to create the required user yourself. |
| `nut_upsmon_password` | Auto-derived | Use this variable to override how the upsmon users **password** is derived. If you override, make sure to create the required user yourself. |
| `nut_upsmon_role` | Auto-derived | Use this variable to override how the upsmon users **role** is derived. If you override, make sure to create the required user yourself. |
| `nut_upsmon_extra` | Multi-line config | Additional upsmon configuratio directly placed in `upsmon.conf` |
| `nut_upsmon_notifycmd` | `undefined` | Path for NOTIFYCMD configuration |
| `nut_upsmon_notifycmd_content` | `undefined` | Content to copy to the notifycmd path |

Refer to the Users Definition section below for details on how to configure users.

### UPS Definition

```yaml
nut_ups:
  - name: UPS
    driver: riello_ups
    device: /dev/ttyUSB0
    description: Some descriptive information
    extra: |
      maxretry = 10
      retrydelay = 1
```

`name` is an arbitrary string that must identify univocally the UPS.

`driver` depends on your hardware and must be one of the [available NUT
driver](http://networkupstools.org/stable-hcl.html). Be sure the NUT
version installed on your server has that specific driver available.

`device` is device where the UPS is listening (typically an USB port or
a serial device).

`description` is optional and is an arbitrary string used for debugging
and reporting purposes.

`extra` is an optional multiline text to be inserted verbatim in the
global section of the relevant configuration file.

Other less used variables, all of them optionals:

    nut_mode: standalone # `man 5 nut.conf`     MODE directive
    nut_powervalue: 1    # `man 5 upsmon.conf`  MONITOR directive, powervalue field
    nut_role: master     # `man 5 upsmon.conf`  MONITOR directive, type field
    nut_services:        # Name of the services to enable
      - nut-server.service
      - nut-monitor.service
      - nut.target

### Users Definition

The NUT users are configured using the `nut_users` variable, see nested scheme below.
You can optionally specify extra configuration snippets that
are added to each user.

The legacy variables `nut_user`, `nut_password` and `nut_role` are now **deprecated**.
For now, the behaviour is as follows:

- If `nut_user` is defined, the legacy variables will be added to `upsd.users` and will be used in `upsmon.conf`
- If `nut_user` is not defined, the first entry of `nut_users` will be used in `upsmon.conf`

This default behaviour can be overriden by explicitely setting 
the `nut_upsmon_*` variables. Note that you are responsible to
create this user in `upsd.users` (for example by adding them to
`nut_users`), the role does not do that automatically.

```yaml
nut_users:
  - name: nutuser1
    password: password1
    role: user
  - name: nutuser2
    password: password2
    role: admin
    extra: |
      sdtype = 2
```

For a detailed description on user attributes that can be set,
please refer to the [`upsd.users` documentation](https://networkupstools.org/docs/man/upsd.users.html).

Example Playbook
----------------

    - hosts: all
      roles:
      - role: ntd.nut
        nut_ups:
          - name: riello
            driver: riello_usb
            device: /dev/ups
            description: iPlug 800

For more examples, please see `tests/test.yml`.

License
-------

MIT

Author Information
------------------

This role was created in 2016 by Nicola Fontana (ntd@entidi.it).
