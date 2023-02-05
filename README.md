# systemd mounts

This role is created upon existent [ypsman
role](https://github.com/ypsman/ansible-systemd-mounts). Setup mounts and
automounts as sysemd Service.

Created mounts can be managed as system services, for example:

```
systemctl status mount-point.mount
systemctl start mount-point.mount
systemctl stop mount-point.mount
```

## Usage

Simple use this role:

```yaml
-
  name: Create mount point
  file:
    state: 'directory'
    path: '{{ backup_mount_point }}'
    mode: '0700'
    owner: 'root'
    group: 'root'
  tags: backup
-
  name: Create systemd NFS mount unit
  include_role:
    name: mikroways.systemd_mounts
    apply:
      tags:
      - backup
  tags: always
```

> Creates a `backup_mount_point` where some NFS server will be mounted. NFS
> options, as other specific mount options are set using `systemd_mounts`
> variable as explained below.


## Options

Mount services are defined as an array of options. Each array entry can set the
following options:

  * `what`: device or service to be mounted.
  * `where`: folder to be used as mount point. It must exists.
  * `type`: mount type.
  * `options`: mount options.
  * `automount`: if this mount will be automounted when accessed.
  * `enabled`: enable this mount point to be started on next boot.
  * `started`: start now this mount serice: mount it now.
  * `target`: list of systemd units to be used as Unit After options. See man
    systemd.unit.
  * `timoutIdleSec`: when automount is used, seconds to umount device. Defaults
    to 30 seconds.
  * `refuseManualStart`: defaults to false but can be changed with this
    options. See man systemd.unit.

### Example

The following example will create two mount services:

```yaml
  backup_mount_point: /var/backups/restic
  systemd_mounts:
    - what: nfs.example.server:/backups-restic
      where: '{{ backup_mount_point }}'
      type: nfs
      options: _netdev,auto
    - where: '{{ backup_mount_point }}'
      automount: true
      enabled: true
      started: true
```

* The first entry creates a NFS mount called `var-backups-restic.mount`. This
  mount service is not mounted on boot, nor started. Only defined.
* The second entry creates an automount service called
  `var-backups-restic.automount`. This one is started and enabled, meaning that
  it starts looking for activity on mount directory to mount the
  previous service.
