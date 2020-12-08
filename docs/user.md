# User Guide

## Installation

### Install
To install the plugin you can use pip. 
```bash
$ pip install git+https://gitlab.cee.redhat.com/ccit/teflo/plugins/teflo_openstack_client_plugin.git@<tagged_version>
```

This will install the [ospclientsdk](https://pypi.org/project/ospclientsdk/) that it is dependent on from pypi

## Credentials
To provision resources with OpenStack requires credentials. You have a couple of options 
to utilize them.

* Export the OpenStack specific environmental variables (they start with **OS_**)

* Provide Teflo the credentials in the `teflo.cfg` and reference that particular 
section using Teflo's `credential` parameter. In most cases Teflo will take the credential
information and export the appropriate OpenStack credential environmental variables for you. 
 
  ```yaml
  #Example teflo scenario referencing the credential in the teflo.cfg
  
   provision:
    - name: ccit_dummy1_trunk
      provisioner: openstack-client
      credential: openstack
      network_trunk:
        parent_port: ccit_dummy_port
        subport:
          - port: ccit_test_port
            segmentation_type: vlan
            segmentation_id: 2007
  ```    
### Configuring Credentials
Below are the provider specific options that can be specified in the `teflo.cfg` 

|key| Description | Required|
|  ---  |   ----  | ---  |
|auth_url|The authentication URL of your OpenStack tenant. (identity)| True|
|tenant_name|The name of your OpenStack tenant.|False|
|username|The username of your OpenStack tenant.|False|
|password|The password of your OpenStack tenant.|False|
|region|The region of your OpenStack tenant to authenticate with.|False|
|domain_name|The name of your OpenStack domain to authenticate with. When not set teflo will use the *default*|False|
|cloud_file|The path to a clouds.yaml file. This is mutually exclusive to the above options.|False|
|cloud|The name of the cloud credential to use when specifying the clouds.yaml. It will default to *openstack*.|False|

```ini
# Example credentials in teflo.cfg using the individual options
[credentials:osp-creds]
auth_url=https://rhos-d.infra.prod.upshift.rdu2.redhat.com:13000
tenant_name=dummy
username=test
password=ChangeMe123
domain_name=example.com

# Example credentials in teflo.cfg using the clouds.yaml file
[credentials:osp-creds]
cloud_file=~/.config/openstack/clouds.yaml
cloud=test
  ```
## Provisioner Configuration


|Setting| Description|
 |  ---  |   ----      |
 |public_network | You can define the network the instance gets the ip from as public to be used later in Ansible inventory |
 |best_available_network| Whether it should check all networks for the best ip availability. Default is *False*|
 |best_available_network_filter| A list of comma separated networks to filter that should be check for ip availability. Default *all*|
 |best_available_network_ip_version| The ip version of the networks to filter the list of networks to check for ip availability | 
 |retry_count| The number of attempts to create/delete asset if an error is encountered. Default *0* | 
 |retry_wait| The amount of time (in seconds) to wait before retry attempts. Default *0* | 
 |unique_name| You can add a unique hash to the name of the asset being provisioned. Default *false* |

```ini
# Example configuration options in teflo.cfg specifying the public_network
[provisioner:openstack-client]
public_network=provider_net_c0

# Example configuration options in teflo.cfg specifying the best_available_network
[provisioner:openstack-client]
best_available_network=True

# Example configuration options in teflo.cfg specifying the best_available_network and filter list
[provisioner:openstack-client]
best_available_network=True
best_available_network_filter=provider_net_cc4,provider_net_cci_8,provider_net_cci_9

# Example configuration option in teflo.cfg using both public_network and best_available_network
[provisioner:openstack-client]
public_network=provider_net_cci_*
best_available_network=True
best_available_network_filter=provider_net_cci_4, provider_net_cci_8, provider_net_cci_9
best_available_network_ip_version=4

# Example configuration option in teflo.cfg configuring retry logic
[provisioner:openstack-client]
retry_attempts=3
retry_wait=60

# Example configuration option in teflo.cfg configuring unique name for the asset
[provisioner:openstack-client]
unique_name=true

  ```

## Provisioning Assets with Openstack-Client Provisioner

The DSL for the provisioner is a pass-thru of the equivalent CLI commands. Which means

### Commands
* The commands are the same except rather the spaces in the commands are replaced with 
  underscrores i.e. network trunk == network_trunk
  
* You don't need to specify the actual create/delete action for the command. The provisioner 
  will infer which action you want to run based on the task being run *provision* or *cleanup*. 
  i.e. you can specify *server* rather than *server_create*
  
* Only the *create*/*delete* actions are supported. The *add*/*delete*/*set*/*unset*/*show*
  actions not supported at this moment.

```yaml
provision:
  - name: db2_dummy
    provisioner: osp-client
    <cli_eqvualent_cmd>: <dict_values>
```
### Options

The options and argument values for the command actions are always defined as dictionaries. Let's
touch on a couple examples

#### Example 1

Any time an option takes an argument
```yaml

# This
command create --option arg


# Can be defined like
command:
  option: arg

```

#### Example 2
Any time an option takes an argument and can be specified multiple times 
```yaml

# This
command create --option arg1 --option arg2

# Can be defined like
command:
  option: 
    - arg1
    - arg2
```

#### Example 3
Any time an option takes an argument with the value in k=v 
```yaml

# This
command create --option arg1=val1

# Can be defined like
command:
  option: 
    - arg1: val1
```

#### Example 4
Any time an option takes an argument with the value in k=v and can be specified multiple times. 
```yaml

# This
command create --option arg1=val1 --option arg2=val2

# Can be defined like
command:
  option: 
    - arg1: val1
    - arg2: val2
```

#### Example 5
Any time an option takes an argument but the value can be a comma separated list of k=v 
```yaml

# This
command create --option arg1=val1,arg2=val2,arg3=val3

# Can be defined like
command:
  option: 
    - arg1: val1
      arg2: val2
      arg3: val3
```

#### Example 6
Any time an option takes no argument and actions like a boolean flag 
```yaml

# This
command create --option

# Can be defined like
command:
  option: True
```
For a full list of commands and options refer to the
[openstackclient](https://docs.openstack.org/python-openstackclient/latest/cli/command-list.html) documentation. 

Now that you understand how to specify the examples. Let's go over some examples. 

### Examples 

#### Example 1
Let's create a server.
```yaml
provision:
  - name: ccit_ci_test_client_a
    groups: client, test_driver
    provisioner: openstack-client
    credential: openstack
    server:
      key_name: test-key
      image: rhel-7.4-server-x86_64-released
      flavor: m1.small
      network:
        - private_network
        - provider_net_cci_8
    ansible_params:
      ansible_user: cloud-user
      ansible_ssh_private_key_file: keys/ccit_key
```

#### Example 2
Let's create 3 servers attached to the provider network.
```yaml
provision:
  - name: ccit_ci_test_client_a
    groups: client, test_driver
    provisioner: openstack-client
    credential: openstack
    server:
      max: 3
      key_name: test-key
      image: rhel-7.4-server-x86_64-released
      flavor: m1.small
      nic:
        - net_id: provider_net_cci_9
    ansible_params:
      ansible_user: cloud-user
      ansible_ssh_private_key_file: keys/ccit_key
```

#### Example 3
Let's create a port and then attach a server to the port
```yaml
provision:
  - name: ccit_test_port
    provisioner: openstack-client
    credential: openstack
    port:
      network: test_private_network

  - name: ccit_ci_test_client_a
    groups: client, test_driver
    provisioner: openstack-client
    credential: openstack
    server:
      key_name: test-key
      image: rhel-7.4-server-x86_64-released
      flavor: m1.small
      port:
        - ccit_test_port
    ansible_params:
      ansible_user: cloud-user
      ansible_ssh_private_key_file: keys/ccit_key

```

#### Example 4
Let's create the ssh key pair and inject it into the instance so we can use it later in the Teflo
Orchestrate task.
```yaml
provision:
    - name: ccit_key
      provisioner: openstack-client
      credential: openstack
      keypair:
        private_key: keys/ccit_key

    - name: ccit_ci_test_client_a
      groups: client, test_driver
      provisioner: openstack-client
      credential: openstack
      server:
        key_name: ccit_key
        image: rhel-7.4-server-x86_64-released
        flavor: m1.small
        network:
          - provider_net_cci_6
      ansible_params:
        ansible_user: cloud-user
        ansible_ssh_private_key_file: keys/ccit_key

```

#### Example 5
Let's create a volume and attach it to the instance.
```yaml
provision:
    - name: test_vol
      provisioner: openstack-client
      volume:
        size: 5
      credential: openstack


    - name: ccit_ci_test_client_a
      groups: client, test_driver
      provisioner: openstack-client
      credential: openstack
      server:
        key_name: test-key
        image: rhel-7.4-server-x86_64-released
        flavor: m1.small
        network:
          - provider_net_cci_6
      server_add_volume:
        device: /dev/vdb
        tgt_res: test_vol
      ansible_params:
        ansible_user: cloud-user
        ansible_ssh_private_key_file: keys/ccit_key

```

#### Example 6
Let's create two volumes and attach it to the instance.
```yaml
provision:
    - name: test_vol
      provisioner: openstack-client
      volume:
        max: 2
        size: 5
      credential: openstack


    - name: ccit_ci_test_client_a
      groups: client, test_driver
      provisioner: openstack-client
      credential: openstack
      server:
        key_name: test-key
        image: rhel-7.4-server-x86_64-released
        flavor: m1.small
        network:
          - provider_net_cci_6
      server_add_volume:
        - device: /dev/vdb
          tgt_res: test_vol-1
        - device: /dev/vdc
          tgt_res: test_vol-2
      ansible_params:
        ansible_user: cloud-user
        ansible_ssh_private_key_file: keys/ccit_key

```

#### Example 7
Let's create two volumes and two instance. Then attach a volume to each instance
```yaml
provision:
    - name: test_vol
      provisioner: openstack-client
      volume:
        max: 2
        size: 5
      credential: openstack


    - name: ccit_ci_test_client_a
      groups: client, test_driver
      provisioner: openstack-client
      credential: openstack
      server:
        max: 2
        key_name: test-key
        image: rhel-7.4-server-x86_64-released
        flavor: m1.small
        network:
          - provider_net_cci_6
      server_add_volume:
        - device: /dev/vdb
          tgt_res: test_vol-1
          res: ccit_ci_test_client_a-1
        - device: /dev/vdc
          tgt_res: test_vol-2
          res: ccit_ci_test_client_a-2
      ansible_params:
        ansible_user: cloud-user
        ansible_ssh_private_key_file: keys/ccit_key

```

**Note**: When provisioning assets that have interdependencies it is best to set the provision
task's concurrency level to `False` so that Teflo can execute in a sequential order. Refer to
the Teflo [Configuration](https://docs.engineering.redhat.com/display/CentralCI/Configure+Teflo)
documentation on how to configure that.

### Ansible Inventory

Teflo can generate Ansible Inventories post provisioning. To have Teflo generate the inventory 
you must specify the following Teflo parameters:

 * `groups` 
 * `ansible_params` 

You should only specify the parameters above for instances. Other
asset types like volumes, network, ports, etc. should not be tagged with those keys to avoid
polluting the inventory file with bogus data. 
    
