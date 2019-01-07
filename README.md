# VMware Instance

Creates a VM by cloning from an existing template in VMware. 

This role supports images that are cloud-init enabled. This role will first query vCenter to see if the instance already exists. If the instance already exists, this role will ensure CPU, memory, etc are correct, but it will not apply the cloud-init configuration. In short, cloud-init will only be run the first time (assuming that option is enabled by setting `vmware_instance_cloudinit: true`).

This role also assumes that all VM names are unique within a vCenter. If you already have an instance with the same name, this role will modify that instance. If you already have multiple instance with the same name, this role will modify the first one found. 

## Requirements

* Pre-staged template to clone from
* Permission to create necessary resources

## Role Variables

|parameter                              |required|default                                 |choices|comments|
|:--------------------------------------|--------|----------------------------------------|-------|--------|
|ansible_python_interpreter             |no      |python                                  |       |This role is normally run on the localhost in a virtual environment. Setting this default variable prevents the error message: <code>Do you have packaging installed? Try 'pip install packaging'- No module named packaging.version</code>. You probably don't want to override this variable unless you know what you're doing.|
|vmware_instance_cluster                |yes     |"mycluster"                             |       |The name of the Vcenter cluster to deploy to.|
|vmware_instance_cpus                   |no      |1                                       |       |The number of CPUs assigned to the VM|
|vmware_instance_datacenter             |yes     |"datacenter"                            |       |The Vcenter datacenter to deploy to|
|vmware_instance_datastore              |yes     |"datastore"                             |       |The Vcenter datastore to deploy to|
|vmware_instance_disk_size              |no      |30                                      |       |The disk size fore the VM|
|vmware_instance_folder                 |yes     |`"/{{ vmware_instance_datacenter }}/vm"`|       |The folder in Vcenter to deploy to|
|vmware_instance_memory                 |no      |1048                                    |       |The amount of memory assigned to the VM|
|vmware_instance_name                   |no      |`"{{ inventory_hostname_short }}"`      |       |The name given to the VM in Vcenter|
|vmware_instance_network                |yes     |"network"                               |       |The network to connect the VM to|
|vmware_instance_template               |yes     |"template"                              |       |The template to use to create the VM|
|vmware_instance_vcenter_fqdn           |yes     |"vcenter.domain.com"                    |       |The fully qualified domain name of the Vcenter|
|vmware_instance_vcenter_password       |yes     |"CHANGEME"                              |       |Password for Vcenter account with permission to deploy VM|
|vmware_instance_vcenter_username       |yes     |"admin'                                 |       |Username for Vcenter account with permission to deploy VM|
|vmware_instance_cloudinit              |no      |false                                   |       |Set this variable to true to use cloud-init to initialize a "cloud-init ready" image|
|vmware_instance_cloudinit_iso_path     |no      |undefined                               |       |This parameter is required when `vmware_instance_cloud_init = true`. Cloud-init on VMware requires uploading an ISO to vCenter with the cloud-init config. This variable is the vCenter datastore path for the ISO that will be uploaded. This ISO will remain after completion of this role. Any file with the same path will be overridden.|
|vmware_instance_cloudinit_iso_datastore|no      |undefined                               |       |This parameter is required when `vmware_instance_cloud_init = true`. The vCenter datastore for the ISO that will be uploaded for cloud-init.|
|vmware_instance_cloudinit_meta_data    |no      |undefined                               |       |This parameter is required when `vmware_instance_cloud_init = true`. A correctly formatted |
|vmware_instance_cloudinit_cloud_config |no      |undefined                               |       |This parameter is required when `vmware_instance_cloud_init = true`.|

## Example Playbook

```yaml
- name: PLAY | Ensure Ubuntu web servers exist
  hosts: [web-servers]
  connetion: local
  roles:
  - role: ar_azure_instance:
    vmware_instance_cluster: "Cluster01"
    vmware_instance_cpus: 2
    vmware_instance_datacenter: "Datacenter01"
    vmware_instance_datastore: "Datastore01"
    vmware_instance_disk_size: 50
    vmware_instance_folder: '/Datacenter01/vm/DevOps/new_automation_vms'
    vmware_instance_memory: 4096
    vmware_instance_name: "{{ inventory_hostname_short | lower }}"
    vmware_instance_network: "public-network"
    vmware_instance_template: "ubuntu-1604-server-02"
    vmware_instance_vcenter_fqdn: "vcenter01.domain.com"
    vmware_instance_vcenter_password: "MYsUP3Rs3CR3TP@ssw0rd!"
    vmware_instance_vcenter_username: "drock.admin"
```

```yaml
- name: PLAY | Ensure Ubuntu web servers exist and run cloud-init to configure instance
  hosts: [web-servers]
  connetion: local
  roles:
  - role: ar_azure_instance:
    vmware_instance_cluster: "Cluster01"
    vmware_instance_cpus: 2
    vmware_instance_datacenter: "Datacenter01"
    vmware_instance_datastore: "Datastore01"
    vmware_instance_disk_size: 50
    vmware_instance_folder: '/Datacenter01/vm/DevOps/new_automation_vms'
    vmware_instance_memory: 4096
    vmware_instance_name: "{{ inventory_hostname_short | lower }}"
    vmware_instance_network: "public-network"
    vmware_instance_template: "ubuntu-1604-server-02"
    vmware_instance_vcenter_fqdn: "vcenter01.domain.com"
    vmware_instance_vcenter_password: "MYsUP3Rs3CR3TP@ssw0rd!"
    vmware_instance_vcenter_username: "drock.admin"
    vmware_instance_cloudinit: true
    vmware_instance_cloudinit_iso_path: "iso/{{ vmware_instance_name }}.iso"
    vmware_instance_cloudinit_iso_datastore: "cloudinit"

    vmware_instance_cloudinit_meta_data:
      instance-id: 'iid-{{ inventory_hostname_short }}-001'
      local-hostname: '{{ inventory_hostname }}'

    vmware_instance_cloudinit_cloud_config:
      manage_resolv_conf: true
      resolv_conf:
        nameservers: ['8.8.8.8', '8.8.4.4']
        searchdomains: ['domain.com', 'test.domain.com']
      yum_repos:
        cloud-init: 
          name: cloud-init
          baseurl: 'http://redhatrepo.domain.com/centos-7-server-rpms'
          enabled: true
          gpgcheck: false
          sslverify: false
      packages: ['lvm2', 'vim', 'net-tools', 'python', 'cifs-utils', 'open-vm-tools']
      runcmd: >-
        [
          'systemctl daemon-reload',
          'systemctl enable vmtoolsd.service',
          'systemctl start --no-block vmtoolsd.service',
          'rm /etc/yum.repos.d/cloud_init.repo',
          'yum clean all',
          'echo > /etc/motd',
        ]
```

## Author Information

|Author              |E-mail                   |
|:-------------------|:------------------------|
|Derek 'dRock' Halsey|derek.halsey@dinohead.com|

## License

MIT License

Copyright (c) 2019 Dinohead LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Role Development Information

### Git SCM
Please refer to the .gitignore file and update accordingly depending on your
development environment, etc.  The particular file was generated at 
gitignore.io and contains settings for the following:
  - Ansible
  - Python
  - Vim
  - Eclipse
  - IntelliJ IDEA
  - Linux
  - Windows
  
### Versioning
Please update VERSION.md as you release new versions of your role and try to
abide by Semantic Versioning (compatibility).

Please consider using Gitflow such that individuals that want to use your role
can identify a version (e.g. master, develop, <tag>, etc.) to use.
  - https://danielkummer.github.io/git-flow-cheatsheet/
  - http://nvie.com/posts/a-successful-git-branching-model/

### Self-contained
Please try to keep this role as self-contained as possible such that it may be
simply installed (e.g. ansible-galaxy install) and applied as part of a 
playbook.
