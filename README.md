# Ansible LVM-Rescript-Backup

This role deploys `lvm-restic-backup.sh`, a bash script to backup locical volumes using restic and rescript.

## Requirements

- restic
- rescript
- pigz
- awk
- pv
- jq
- python3-humanfriendly

## Example Playbook

```yaml
---

- name: LVM Rescript backup-filelevel
  hosts: hypervisors
  become: true

  roles:
    - sebastian13.restic
    - sebsatian13.rescript
    - sebastian13.lvm-rescript-backup

  vars:
    restic_repos:
      - name: example-filelevel
        restic_repo: 's3:https://...'
        restic_password: !vault |
              $ANSIBLE_VAULT;1.1;AES256
              65616131343239383...36333833432393830
        restic_aws_id: !vault |
              $ANSIBLE_VAULT;1.1;AES256
              33326165343464663...64306164643562363
        restic_aws_key: !vault |
              $ANSIBLE_VAULT;1.1;AES256
              32356139333035363...33665666403232353
        rescript_email: "...@example.com"
      - name: usb-blocklevel-gz
        restic_password: !vault |
              $ANSIBLE_VAULT;1.1;AES256
              65616131343239383...66462378322393830
        restic_repo: '/mnt/usb/backup/restic/usb-blocklevel-gz'

    rescript_cronjobs:
      - name: example-filelevel-cleanup
        repo_name: 'example-filelevel'
        cron_hour: '18'
        cron_weekday: 'SUN'
        rescript_command: 'cleanup --email --log'

    lvm_rescript:
      - name: example-filelevel
        LVM_BACKUP_TYPE: "file-level-backup"
        LVM_BACKUP_LIST: "/etc/restic/backup-filelevel.txt"
        LVM_CRON_HOUR: '22'
        LVM_CRON_WEEKDAY: 'MON-SAT'
        LVM_HEALTHCHECK_URL: 'https://healthchecks.example.com/ping/...'
      - name: usb-blocklevel-gz
        LVM_BACKUP_USB: true
        LVM_CHECK: true
        LVM_BACKUP_TYPE: 'block-level-gz-backup'
        LVM_BACKUP_LIST: '/etc/restic/backup-blocklevel-gz-usb.txt'
        LVM_CRON_STATE: 'absent'
        LVM_HEALTHCHECK_URL: 'https://healthchecks.example.com/ping/...'
```

Passwords and secret keys should be encrypted using ansible-vault

```bash
ansible-vault encrypt_string --stdin-name 'RESTIC_PASSWORD'
```

The playbook must then be run as follows:

```bash
ansible-playbook lvm-rescript-backup.yml --ask-vault-pass
```
