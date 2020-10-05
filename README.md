Ansible Role: Proxmox VM
=========

The role creates VMs using a template created from cloud-init image.

Requirements
------------

The role requires `running proxmox server` with installed `proxmoxer` (pip package).

Role Variables
--------------

Available variables are listed below along with default values (`defaults/main.yml`):

    proxmox_host: proxmox.example.com
    proxmox_user: root@pam
    proxmox_password: secret

These are using to log into the Proxmox API.

    proxmox_vms:
      app-1.example.com:
        node: pve
        vmid: 100
        sockets: 1
        cores: 4
        memory: 4096
        onboot: yes
        # cloud-init settings
        user: ansible
        password: <plaintext>
        ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}
        dns:
          - 1.1.1.1
          - 8.8.8.8
        domain: example.com
        ip: 192.168.0.1/24
        gateway: 192.168.0.254

All available setting of VM are listed above.

`app-1.example.com`

That specifies the VM name which only using on web interface.

`node: pve`

Proxmox VE node where the new VM will be created. (default: yes)

`vmid: 100`

Specifies the VM ID.

`sockets: 1`

Sets the number of CPU sockets. (default: 1)

`cores: 4`

Specify number of cores per socket. (default: 2)

`memory: 4096`

Memory size in MB for instance. (default: 1024)

`onboot: yes`

Specifies whether a VM will be started during system bootup. (default: yes)

`user: ansible`

User name to change ssh keys and password for instead of the image’s configured default user.

`password: secret`

Password to assign the user.

`ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"`

Setup public SSH keys (one key, OpenSSH format).

    dns:
      - 1.1.1.1
      - 8.8.8.8

Sets DNS server IP address for a VM.

`domain: example.com`

Sets DNS search domains for a VM.

`ip: 192.168.0.1/24`

Specify IP address for net0 interface. IP addresses use CIDR notation.

`gateway: 192.168.0.254`

Specify gateway for net0 interface.


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
          # minimum options
          app-2.example.com:
            vmid: 101

      roles:
         - geerlingguy.pip
         - rantanevich.proxmox_vm

License
-------

MIT
