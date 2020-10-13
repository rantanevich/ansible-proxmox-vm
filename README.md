Ansible Role: Proxmox VM
=========

The role creates VMs using a template created from cloud-init image.

Requirements
------------

The role requires `running proxmox server` with installed `proxmoxer` (pip package).

Role Variables
--------------

Variables for logging into Proxmox server:

| Variable             | Default   | Comments                                          |
|----------------------|-----------|---------------------------------------------------|
| proxmox_api_host     | localhost | Specify the target host of the Proxmox VE cluster | 
| proxmox_api_user     | root@pam  | Specify the user to authenticate with.            |
| proxmox_api_password | secure    | Specify the password to authenticate with.        |

In example below listed all variables for setting VMs:

    proxmox_vms:
      - name: app-1.example.com
        node: pve
        vmid: 100
        sockets: 1
        cores: 4
        memory: 4096
        onboot: yes
        volumes:
          - storage: local-lvm
            format: raw
            size: 100G
          - storage: local-lvm
            format: raw
            size: 1000G
        # cloud-init settings
        user: ansible
        password: <plaintext>
        ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
        dns:
          - 1.1.1.1
          - 8.8.8.8
        domain: example.com
        ip: 192.168.0.1/24
        gateway: 192.168.0.254

More detailed:

| Variable | Required  | Default | Comments                                                 |
|----------|-----------|---------|----------------------------------------------------------|
| name     | yes       | none    | Specifies the name of the VM                             |
| node     | yes       | pve     | Specifies Proxmox node, where the new VM will be created |
| vmid     | yes       | none    | Specifies the VM ID. It must be unique                   |
| sockets  | no        | 1       | Sets the number of CPU sockets (1-N)                     |
| cores    | no        | 2       | Specifies number of cores per socket                     |
| memory   | no        | 1024    | Specifies memory size in MB for instance                 |
| onboot   | no        | yes     | Specifies if a VM will be started during system bootup   |

Volumes are created and attached to buses in the same order (start from 1).

| Variable | Required  | Default              | Comments                                                                                     |
|----------|-----------|----------------------|----------------------------------------------------------------------------------------------|
| storage  | yes       | none                 | The storage identifier.                                                                      |
| size     | yes       | none                 | Size in kilobyte (1024 bytes). Optional suffixes M (megabyte, 1024K) and G (gigabyte, 1024M) |
| format   | no        | specified by storage | Format of disk image                                                                         |

Cloud Init settings:

| Variable | Required  | Example                             | Comments                                                 |
|----------|-----------|-------------------------------------|----------------------------------------------------------|
| user     | no        | ansible                             | User will be created instead of the image’s default user |
| password | no        | qwerty                              | Password to assign the user                              |
| ssh_key  | no        | lookup('file', '~/.ssh/id_rsa.pub') | Setup public SSH keys (OpenSSH format)                   |
| dns      | no        | ['1.1.1.1', '8.8.8.8']              | Sets DNS server IP address                               |
| domain   | no        | example.com                         | Sets DNS search domains                                  |
| ip       | no        | 192.168.0.1/24                      | Specify IP address for the corresponding interface       |
| gateway  | no        | 192.168.0.254                       | Specify gateways for the corresponding interface         |


Dependencies
------------

This role need to installed proxmoxer (pip package) on proxmox server. I usually use `geerlingguy.pip` role for that.

    roles:
      - geerlingguy.pip
        pip_install_packages:
          - name: proxmoxer

Example Playbook
----------------

    - hosts: proxmox
      vars:
        pip_install_packages:
          - name: proxmoxer
        proxmox_api_host: <dns or ip address of proxmox server>
        proxmox_api_user: root@pam 
        proxmox_api_password: <plaintext password>
        proxmox_vms:
          # all options
          app-1.example.com:
            node: pve
            vmid: 100
            sockets: 1
            cores: 4
            memory: 4096
            onboot: yes
            volumes:
              - storage: local-lvm
                format: raw
                size: 10G
              - storage: remote
                format: qcow2
                size: 50G
            user: ansible
            password: <plaintext>
            ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
            dns:
              - 1.1.1.1
              - 8.8.8.8
            domain: example.com
            ip: 192.168.0.1/24
            gateway: 192.168.0.254
          # minimum options
          app-2.example.com:
            vmid: 101

      roles:
         - geerlingguy.pip
         - rantanevich.proxmox_vm

License
-------

MIT
