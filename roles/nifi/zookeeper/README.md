# Apache ZooKeeper

![Lint Code Base] ![Molecule]

Ansible role for installing and configuring Apache ZooKeeper

This role can be used to install and cluster multiple ZooKeeper nodes, this uses
all hosts defined for the "zookeeper-nodes" group in the inventory file by
default. All servers are added to the zoo.cfg file along with the leader and
election ports. Firewall ports could be opened after setting `true` to
`zookeeper_firewalld` variable

## Supported Platforms

- Debian 10.x
- RedHat 7
- RedHat 8
- Ubuntu 18.04.x
- Ubuntu 20.04.x

## Requirements

Java: Java 8 / 11

Ansible 2.9.16 or 2.10.4 are the minimum required versions to workaround an
issue with certain kernels that have broken the `systemd` status check. The
error message "`Service is in unknown state`" will be output when attempting to
start the service via the Ansible role and the task will fail. The service will
start as expected if the `systemctl start` command is run on the physical host.
See <https://github.com/ansible/ansible/issues/71528> for more information.

## Role Variables

| Variable                                 | Default                                                           | Comment                                                        |
| ---------------------------------------- | ----------------------------------------------------------------- | -------------------------------------------------------------- |
| zookeeper_mirror                         | <https://dlcdn.apache.org/zookeeper>                              |                                                                |
| zookeeper_version                        | 3.9.2                                                             |                                                                |
| zookeeper_package                        | apache-zookeeper-{{ zookeeper_version }}-bin.tar.gz               |                                                                |
| zookeeper_group                          | zookeeper                                                         |                                                                |
| zookeeper_user                           | zookeeper                                                         |                                                                |
| zookeeper_root_dir                       | /usr/share                                                        |                                                                |
| zookeeper_install_dir                    | '{{ zookeeper_root_dir}}/apache-zookeeper-{{zookeeper_version}}'  |                                                                |
| zookeeper_dir                            | '{{ zookeeper_root_dir }}/zookeeper'                              |                                                                |
| zookeeper_log_dir                        | /var/log/zookeeper                                                |                                                                |
| zookeeper_data_dir                       | /var/lib/zookeeper                                                |                                                                |
| zookeeper_data_log_dir                   | /var/lib/zookeeper                                                |                                                                |
| zookeeper_client_port                    | 2181                                                              |                                                                |
| zookeeper_id                             | 1                                                                 | Unique per server and should be declared in the inventory file |
| zookeeper_leader_port                    | 2888                                                              |                                                                |
| zookeeper_election_port                  | 3888                                                              |                                                                |
| zookeeper_servers                        | zookeeper-nodes                                                   | See below                                                      |
| zookeeper_servers_use_inventory_hostname | false                                                             | See below                                                      |
| zookeeper_environment                    | "JVMFLAGS": "-javaagent:/opt/jolokia/jolokia-jvm-1.6.0-agent.jar" |                                                                |
| zookeeper_config_params                  |                                                                   | A key-value dictionary that will be templated into zoo.cfg     |
| zookeeper_firewalld                      | false                                                             |                                                                |

## Inventory and zookeeper_servers variable

zookeeper_servers variable above accepts a list of inventory hostnames. These
will be used in the `zoo.cfg` to configure a multi-server cluster so the hosts
can find each other. By default, the hostname used in the `zoo.cfg` will be the
hostname reported by the `hostname` command on the server(provided by the
`ansible_nodename` variable). See the example below.

Assuming the below inventory file, and that the `hostname` command returns only
the hostname and does not include the domain name.

```ini
[zookeeper-nodes]
zoo1.foo.com zookeeper_id=1       #hostname command returns "zoo1"
zoo2.foo.com zookeeper_id=2       #hostname command returns "zoo2"
zoo3.foo.com zookeeper_id=3       #hostname command returns "zoo3"
```

And assuming the following role variables:

```yaml
---
- role: sleighzy.zookeeper
  zookeeper_servers:
    - zoo1.foo.com
    - zoo2.foo.com
    - zoo3.foo.com
```

The templated `zoo.cfg` file will contain the below entries:

```ini
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
```

If you DO NOT want this behaviour and would like the `zoo.cfg` to template the
inventory_hostname then set `zookeeper_servers_use_inventory_hostname` to `true`

### Default Ports

| Port | Description                         |
| ---- | ----------------------------------- |
| 2181 | Client connection port              |
| 2888 | Quorum port for clustering          |
| 3888 | Leader election port for clustering |

### Default Directories and Files

| Description                                | Directory / File                            |
| ------------------------------------------ | ------------------------------------------- |
| Installation directory                     | `/usr/share/apache-zookeeper-<version>`     |
| Symlink to install directory               | `/usr/share/zookeeper`                      |
| Symlink to configuration                   | `/etc/zookeeper/zoo.cfg`                    |
| Log files                                  | `/var/log/zookeeper`                        |
| Data directory for snapshots and myid file | `/var/lib/zookeeper`                        |
| Data directory for transaction log files   | `/var/lib/zookeeper`                        |
| Systemd service                            | `/usr/lib/systemd/system/zookeeper.service` |
| System Defaults                            | `/etc/default/zookeeper`                    |

## Starting and Stopping ZooKeeper services

- The ZooKeeper service can be started via: `systemctl start zookeeper`
- The ZooKeeper service can be stopped via: `systemctl stop zookeeper`

## Four Letter Word Commands

ZooKeeper can use commands based on four letter words, see
<https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_4lw>

The below example uses the stat command to find out which instance is the leader
:

```bash
for i in 1 2 3 ; do
  echo "zookeeper0$i is a "$(echo stat | nc zookeeper0$i 2181 | grep ^Mode | awk '{print $2}');
done
```

## Dependencies

No dependencies

## Example Playbook

```yaml
- hosts: zookeeper-nodes
  roles:
    - nifi.zookeeper
```
