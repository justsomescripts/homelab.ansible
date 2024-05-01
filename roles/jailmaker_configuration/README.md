# Role -  jailmaker_configuration

Setup [Jailmaker](https://github.com/Jip-Hop/jailmaker/) jails on a TrueNAS scale system

## 📋 Requirements

* [ansible.builtin.stat](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/stat_module.html)
* [ansible.builtin.meta](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/meta_module.html)
* [ansible.builtin.debug](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)
* [ansible.builtin.get_url](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html)
* [ansible.builtin.template](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)
* [ansible.builtin.service](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html)
* [ansible.builtin.command](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html)

## Prerequisites

| name                                                        | type           | required | choices                     | default                                                                                                                                                                                                                                                    | description                                                            |
| ----------------------------------------------------------- | -------------- | -------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `jailmaker_configuration__jail.name`                        | string         | ✅       | Jailmaker Jail name         | `debian`                                                                                                                                                                                                                                                   | name of the Jail                                                       |
| `jailmaker_configuration__jail.dataset`                     | string         | ✅       | Jailmaker dataset           | `tank`                                                                                                                                                                                                                                                     | name of the dataset jailmaker is installed on                          |
| `jailmaker_configuration__os.autostart`                     | bool           | ❌       |                             | `true`                                                                                                                                                                                                                                                     | start Jail at boot / Jailmaker startup                                 |
| `jailmaker_configuration__os.seccomp_enabled`               | bool           | ❌       |                             | `true`                                                                                                                                                                                                                                                     | enable Seccomp                                                         |
| `jailmaker_configuration__os.distribution.name`             | string         | ✅       | supported LXC image         | `debian`                                                                                                                                                                                                                                                   | name of the distribution                                               |
| `jailmaker_configuration__os.distribution.release`          | string         | ✅       | supported LXC image         | `bookworm`                                                                                                                                                                                                                                                 | release version of the distribution                                    |
| `jailmaker_configuration__os.gpu_passtrough.intel_enabled`  | bool           | ❌       |                             | `false`                                                                                                                                                                                                                                                    | enable Intel GPU passtrough                                            |
| `jailmaker_configuration__os.gpu_passtrough.nvidia_enabled` | bool           | ❌       |                             | `false`                                                                                                                                                                                                                                                    | enable Nvidia GPU passtrough                                           |
| `jailmaker_configuration__mounts`                           | array of dicts | ❌       | host bind mounts            | `[]`                                                                                                                                                                                                                                                       | host bind mounts (requires `source`=host and `destination`=Jail paths) |
| `jailmaker_configuration__systemd.nspawn.user_args`         | array          | ❌       | systemd-nspawn user args    | `- resolv-conf=bind-host`<br>`- system-call-filter='add_key keyctl bpf'`                                                                                                                                                                                   | systemd-nspawn flags passed to Jailmaker config                        |
| `jailmaker_configuration__systemd.nspawn.default_args`      | array          | ❌       | systemd-nspawn default args | `- keep-unit`<br>`- quiet`<br>`- boot`<br>`- bind-ro=/sys/module`<br>`- inaccessible=/sys/module/apparmor`                                                                                                                                                 | systemd-nspawn flags passed to Jailmaker config                        |
| `jailmaker_configuration__systemd.run.default_args`         | array          | ❌       | systemd-nspawn default args | `- property=KillMode=mixed`<br>`- property=Type=notify`<br>`- property=RestartForceExitStatus=133`<br>`- property=SuccessExitStatus=133`<br>`- property=Delegate=yes`<br>`- property=TasksMax=infinity`<br>`- collect`<br>`- setenv=SYSTEMD_NSPAWN_LOCK=0` | systemd-nspawn flags passed to Jailmaker config                        |

## 💻 Example Usage

```yaml
---
# host_vars
truenas_jails:
  - jailmaker_configuration__jail:
      name: testjail
      dataset: speed
    jailmaker_configuration__mounts:
      - source: /mnt/speed/jailmaker-data/container/dockge/data
        destination: /opt/container/data/dockge
      - source: /mnt/speed/jailmaker-data/container/dockge/stacks
        destination: /opt/container/stacks

---
# Playbook
- name: TrueNAS SCALE Jails
  hosts: truenas
  remote_user: "{{ remote_user_configuration__admin.user }}"
  tasks:

    - name: Setup container environment
      tags:
        - tn_ctr
      block:

        - name: Setup Debian Jail
          ansible.builtin.import_role:
            name: jailmaker_configuration
          tags:
            - tn_jailmaker_configuration
          vars: "{{ item }}"
          loop: "{{ truenas_jails }}"

...
```

## 📜 License

See `LICENSE`

## ✍️ Author

David Gries <<mail@dgries.de>>