A playbook that implements backup through [borgmatic](https://torsion.org/borgmatic/).

Sample usage for a nextcloud backup playbook:

```yaml
---
- name: Setup backup for gitea
  import_playbook: "configure_borgmatic.yml"
  vars:
    hostlist: nextcloud
    service_name: "nextcloud"
    backup_paths:
      - /srv/nextcloud/data  # Specify the path to backup
    borg_repo_paths:
      - "/mnt/root-only/{{ service_name }}_bkp"
    borg_remote:
      ssh_key: "/path/to/rsync.net/key"
      path: "ssh://username@domain.rsync.net/./{{ service_name }}_bkp"
    hooks:
      healthchecks: "https://hc-ping.com/5faa7e74-4054-442d-8a61-82862b840c71"
    nfs:
      server: "nas"  # hostname of an NFS server
      description: "Backup mount for {{ service_name }}"
      what: "/volume1/{{ service_name }}_backup"  # name of the remote share
      where: "{{ borg_repo_paths[0] }}"  # where to mount it
      options: "vers=4.1,_netdev,rw,noatime,noauto,soft"  # NFS options
    db:
      name: "nextcloud"  # name of the database
      schema: "nextcloud"  # name of the schema
      hostname: "localhost"  # where DB is located
```

The playbook implements 3-2-1 backups by creating two separate borg backups into two remotes. One remote is on a local NFS server, the other on rsync.net. The playbook can also ping [healthchecks.io](https://healthchecks.io/) to alert of failures/timing out backups.

The `configure_borgmatic.yml` playbook will:

1. Generate a passphrase for the borg repository using diceware password generator
2. Create a postgresql user with read-only access to specified database
3. Create a mount for a remote server's NFS share
4. Create required systemd units
5. Place borgmatic config where the unit expects it
