Beats
=========

A role to install and manage all supported beats offered from Elastic. This is a generic role that will install any combination of beats agents.
I have confirmed this role to work with packetbeat, auditbeat, filebeat, and metricbeat on the supported platforms listed
on the Galaxy page. In concept, this play *should* work for all beats agents but small idiosyncrasies may break it.

Requirements
------------

Install the jmespath python module before using this role.

Role Variables
--------------

User required:
  - **beat_config**.*<beat name>*: The path to the custom configuration for the installed agents. <beat name> should be one of the supported beats agent names. Default: undefined.

Defaults:
  - **beat_version**: Set the installed version. This will apply to all beats installed on a host. Default: 7.4.0
  - **beat_user**: Set which user the agent should run as and the owner of all configuration files. Default: root
  - **beat_group**: Set with group the agent should run under and the group of all configuration files. Default: root
  - **beat_config_mode**: Set the permission mode for all configuration files. Default: 0600
  - **beat_inventory_file**: File location to write the list of installed beats. This is used to determine when to uninstall beats. Default: /var/log/installed_heats.txt

Variables:
  - **installed_beats**: Internally used list to capture what beats were installed/configured each run.
  - **package_and_service_name**.*<beat name>*: Dictionary of beat names mapped to their package names.

Dependencies
------------

N/A

Example Playbook
----------------

It is possible to configure the role in the playbook.
```yml
---
# site.yml
- hosts: servers
  become: yes
  roles:
  - { 
    role: lksnyder0.beat,
    beats: [
      "filebeat",
      "metricbeat"
    ],
    beat_config: {
      filebeat: {
        config: "files/filebeat/filebeat.yml",
        modules: [
          {
            name: "system"
          }
        ]
      },
      metricbeat: {
        config: "files/metricbeat/metricbeat.yml",
        modules: [
          {
            name: "system",
            config: "files/metricbeat/modules.d/system.yml"
          }
          ,{
            name: "beat"
            # ,config: "files/metricbeat/modules.d/beat.yml"
          }
        ]
      }
    } }
```

However, I prefer to configure this in the inventory file to keep the role file clean. It also allows 
for per-server configuration. The example below will install filebeat on server1 and filebeat and
metricbeat on server2 with a common configuration between them.
```yml
---
# inventory.yml
all:
  children:
    servers:
      vars:
        beat_config:
          filebeat:
            config: "files/filebeat/filebeat.yml"
            modules:
            - name: system
          metricbeat:
            config: "files/metricbeat/metricbeat.yml"
            modules: 
            - name: system
              config: "files/metricbeat/modules.d/system.yml"
            - name: beat
      hosts:
        server1:
          beats:
          - filebeat
        server2:
          beats:
            - filebeat
            - metricbeat
```
```yml
---
# site.yml
- name: Install beats
  hosts: servers
  roles:
  - {
    role: "lksnyder0.beat"
  }
```

License
-------

[BSD 2-Clause (License FreeBSD/Simplified)](https://tldrlegal.com/l/freebsd)

TODO
----

* Add task to manage keystore values.
