# Ansible LVM-Rescript-Backup

### How to use?

1. Create a playbook `lvm-rescript-backup.yml`

	```yaml
	---
	- hosts: hypervisors
	  become: true

	  tasks:
	    - name: Install Restic
	      ansible.builtin.import_role:
	      	name: restic

	    - name: Install Rescript
	      ansible.builtin.import_role:
	      	name: rescript

	    - name: Setup LVM-Rescript-Backup
	      ansible.builtin.import_role:
	      	name: lvm-rescript-backup
	```

1. Define `restic_repos:` & `lvm_rescript:` in a host_vars file.

	```yaml
	restic_repos:
	  - name: nas00-filelevel
	    RESTIC_REPO: 's3:https://asdf.at:123456/mybucket'
	    RESTIC_PASSWORD: 'secure'
	    AWS_ID: 'asdf'
	    AWS_KEY: 'secure'
	    CRON_STATE: 'absent'
	lvm_rescript:
	  - name: nas00-filelevel
	    LVM_BACKUP_TYPE: "file-level-backup"
	    LVM_BACKUP_LIST: "/etc/restic/backup-filelevel.txt"
	    LVM_CRON_HOUR: '22'
	    LVM_CRON_WEEKDAY: 'MON-SAT'
	    LVM_HEALTHCHECK_URL: 'https://healthchecks.example.com/123456'
	```

1. Think about encrypting the passwords:

   ```bash
   ansible-vault encrypt_string --stdin-name 'RESTIC_PASSWORD'
   ```

1. Run the playbook

	```bash
	ansible-playbook lvm-rescript-backup.yml --ask-vault-pass

	# Deploy to certain servers & start at a certain task
	ansible-playbook lvm-rescript-backup.yml --limit xen00 --start-at-task "Deploy lvm backup scripts"
	```
