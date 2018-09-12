|# vmware_instance| | | | |

Creates a VM by cloning from an existing template in VMware.

## Requirements

* Pre-staged template to clone from
* Permission to create necessary resources

## Role Variables

|parameter|required|default|choices|comments|
|---|---|---|---|---|
|ansible_python_interpreter|no|python| |This role is normally run on the localhost in a virtual environment. Setting this default variable prevents the error message: <code>Do you have packaging installed? Try 'pip install packaging'- No module named packaging.version</code>. You probably don't want to override this variable unless you know what you're doing.|
|vmware_instance_cluster|yes|"mycluster"| |The name of the Vcenter cluster to deploy to.|
|vmware_instance_cpus|no|1| |The number of CPUs assigned to the VM|
|vmware_instance_datacenter|yes|"datacenter"| |The Vcenter datacenter to deploy to|
|vmware_instance_datastore|yes|"datastore"| |The Vcenter datastore to deploy to|
|vmware_instance_disk_size|no|30| |The disk size fore the VM|
|vmware_instance_folder|yes|'/SDC/vm'| |The folder in Vcenter to deploy to|
|vmware_instance_memory|no|1048| |The amount of memory assigned to the VM|
|vmware_instance_name|no|<code>"{{ inventory_hostname_short }}"</code>| |The name given to the VM in Vcenter|
|vmware_instance_network|yes|"network"| |The network to connect the VM to|
|vmware_instance_template|yes|"template"| |The template to use to create the VM|
|vmware_instance_vcenter_fqdn|yes|"vcenter.domain.com"| |The fully qualified domain name of the Vcenter|
|vmware_instance_vcenter_password|yes|"CHANGEME"| |Password for Vcenter account with permission to deploy VM|
|vmware_instance_vcenter_username|yes |"admin'| |Username for Vcenter account with permission to deploy VM|
## Dependencies

|Role|Description|Source|
|---|---|---|
|TBD|TBD|TBD|

## Example Playbook


```yaml
- name: PLAY | Ubuntu web servers exist
  hosts: [web-servers]
  connetion: local
  roles:
  - role: ar_azure_instance:
    vmware_instance_cluster: "General-SDC"
    vmware_instance_cpus: 2
    vmware_instance_datacenter: "SDC"
    vmware_instance_datastore: "Tintri-SDC-NFS-05"
    vmware_instance_disk_size: 50
    vmware_instance_folder: '/SDC/vm/DevOps/new_automation_vms'
    vmware_instance_memory: 4096
    vmware_instance_name: "{{ inventory_hostname_short | upper }}"
    vmware_instance_network: "'Server4-10.251.14.0%2f23-VLAN232'"
    vmware_instance_template: "ubuntu-1604-server-02"
    vmware_instance_vcenter_fqdn: "knevcsiwp001.kiewitplaza.com"
    vmware_instance_vcenter_password: "MYsUP3Rs3CR3TP@ssw0rd!"
    vmware_instance_vcenter_username: "drock.admin"
```

## License

TBD

## Author Information

|Author|E-mail|
|---|---|
|Derek 'dRock' Halsey|derek.halsey@kiewit.com|

## Role Development Information

### Contributing

1. Fork it
1. Create your feature branch (`git checkout -b my-new-feature`)
1. Commit your changes (`git commit -am 'Add some feature'`)
1. Push to the branch (`git push origin my-new-feature`)
1. Create new Pull Request

### Git SCM
Please refer to the .gitignore file and update accordingly depending on your
development environment, etc.  The particular file was generated at 
[gitignore.io](https://www.gitignore.io/) and contains settings for the following:
  - Ansible
  - Python
  - Vim
  - Eclipse
  - IntelliJ IDEA
  - Linux
  - Windows
  
### Versioning
Please update [VERSION.md](./VERSION.md) as you release new versions of your role and try to
abide by [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

### Self-contained
Please try to keep this role as self-contained as possible such that it may be
simply installed (e.g. `ansible-galaxy install`) and applied as part of a 
playbook.
