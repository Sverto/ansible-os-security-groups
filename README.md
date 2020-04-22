# OpenStack Security Groups Role
OpenStack API wrapper. Can currently be used to:
- Configure self-defined security groups
- Or you can import `../openstack/vars` in your own role which contains a template of the cloud authentication object and create your own `os` tasks (see example below)


## Requirements
Openstack client on Ansible server:
```python
# Upgrade PIP
sudo pip install --upgrade pip
# Upgrade Python decorator to fix SDK not being detected by Ansible
sudo pip install --upgrade decorator
# Install OpenStack Shade Client and other dependencies
sudo pip install ndg-httpsclient python-novaclient python-keystoneclient shade
# Install Openstack client (used because Ansible itself hasn't full OpenStack support yet)
sudo pip install python-openstackclient
openstack --version
```
To workarround proxy issues you can add the following to every pip command:
- --trusted-host pypi.org --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --proxy http://myproxy:8080


## Tags
| Tag                | Description                                            | About                              |
| ------------------ | ------------------------------------------------------ | ---------------------------------- |
| os_server_facts    | Gather Openstack host facts on var `os_server_facts[]` | Never ran by default, optional     |
| os_security_groups | Create user defined security groups                    | Uses value of `os_security_groups` |


## Parameters
Required parameters with example values:
```yaml
# Openstack URL
os_url: https://myopenstackinstance:13000/v3/

# Openstack API user
os_user: openstackuser
os_password: openstackpassword
os_user_domain: user.domain

# Set default Openstack project name and domain
os_project: my-project-name
os_project_domain: project.domain
```

Optional parameters:
```yaml
# Optional: Only apply security groups of the set environment
os_environment: ""
```

Define security groups:
```yaml
os_security_groups: [ # Array of security groups
  # Security group example
  { 
    name:           "sec-group name",        # Required
    description:    "sec-group description", # Optional
    project:        "",                      # Optional (defaults to os_project)
    project_domain: "",                      # Optional (defaults to var os_project_domain)
    # Optional: Define rules for security group:
    rules: [{ 
      direction: "",        # Defaults to ingress
      protocol:  "",        # Defaults to tcp
      ethertype: "",        # Defaults to IPv4
      port_range_min: 8080, # Use the "port" var instead of "port_range_min" and "port_range_max" if setting a single port. Is required depends on "protocol" set
      port_range_max: 0,    # Defaults to port_range_min var
      ip: ""                # Defaults to 0.0.0.0/0
    }],
    # Optional: Define the hosts on which the security group should be applied (default is inventory hosts):
    hosts: [
      'fqdnhost1',
      'fqdnhost2'
    ],
    # Optional: Bind to environment (creation of this security group is skipped when not equal to os_environment):
    environment: int
  }
]
```

See also [vars\main.yml](vars\main.yml) for more detail about the cloud authentication object used in OS tasks.


## Example Playbook
```yaml
- hosts: openstack
  gather_facts: no # We can't gather facts if we don't have a flow yet from Ansible to the target
  roles:
    - { role: sverto.os_security_groups, tags: os }
```


## Example Inventory
```yaml
openstack:
  hosts:
    webserver1.openstack.domain:
    webserver2.openstack.domain:
  vars:
    os_url: https://myopenstackinstance:13000/v3/
    os_user: openstackuser
    os_password: openstackpassword
    os_user_domain: user.domain
    os_project: my-project-name
    os_project_domain: project.domain
    os_security_groups: [
    {
      name: "Web Access",
      description: "Open web access. Created using Ansible.",
      rules: [
        { port: 8080, ip: "10.0.0.0/16" },
        { port: 8080, ip: "20.1.2.3/32" }
      ]
    }]
```


## Using the cloud authentication object in your own role
```yaml
# Make sure the Openstack required vars are set
- set_facts:
    os_url: https://myopenstackinstance:13000/v3/
    os_user: openstackuser
    os_password: openstackpassword
    os_user_domain: user.domain
    os_project: my-project-name
    os_project_domain: project.domain
# Then import the cloud authentication templater
- name: Get cloud authentication object
  include_vars:
   dir: ../openstack/vars
# Use the templated "cloud_provider" var in your own "os_*" tasks
- name: My own OS task
  os_security_group:
    state: present
    cloud: '{{ cloud_provider }}' # Used here
    name: My security group name
    description: A test example
```

## Author
Sverto
