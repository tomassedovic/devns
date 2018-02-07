# devns

An unsupported nsupdate-compatible DNS that could be used with
the OpenStack provisioning playbooks in openshift-ansible.

This project is **NOT suitable for production**. Its intention is to ease the
OpenShift and OpenStack integration development and testing.


## How to use this

### Setup

You need to decide on the domain name for your OpenShift cluster. That domain
must be same here and in the openshift-ansible inventory.

The default for both is `openshift.example.com`.


```
$ git clone https://github.com/tomassedovic/devns.git
$ cd devns
$ vi vars.yaml
# Set: dns_domain
# Set: key_name
# Set: image
# Set: flavor
```

The playbook has only been tested on CentOS 7, but it should work on
RHEL 7 as well.


### Deploy

Run this command:

```
source keystonerc  # Or whatever your OpenStack credentials file is called
ansible-playbook deploy.yaml -e @vars.yaml
```

**NOTE:** if your SSH key is not `~/.ssh/id_rsa`, you need to pass
`--private-key path/to/key` to `ansible-playbook`.

**NOTE:** if your image's SSH user (the cloud-init username) is not
`centos`, you need to pass `--user username` to `ansible-playbook`.

After the playbook finishes, note the "Print configured keys" task. It should
look something like this:

```
TASK [infra-ansible/roles/config-dns-server : Print configured keys - if requested] ***********************************************************************************************************
Monday 29 January 2018  18:21:53 +0100 (0:00:01.437)       0:01:36.860 ********
ok: [10.40.128.128 -> 10.40.128.128] => {
    "nsupdate_keys": {
        "public-openshift.example.com": {
            "key_algorithm": "HMAC-SHA256",
            "key_secret": "YuVcVQsLjKRX7qy9PKSwIdrroERVmS/3nSH0DyyzBU8="
        }
    }
}

```

### Use in openshift-ansible

Edit your `inventory/group_vars/all.yml` and put the following in:

```
openshift_openstack_dns_nameservers: [10.40.128.128]
openshift_openstack_public_hostname_suffix: "-public"
openshift_openstack_external_nsupdate_keys:
  private:
    key_secret: "YuVcVQsLjKRX7qy9PKSwIdrroERVmS/3nSH0DyyzBU8="
    key_algorithm: "HMAC-SHA256"
    key_name: "public-openshift.example.com"
    server: "10.40.128.128"
  public:
    key_secret: "YuVcVQsLjKRX7qy9PKSwIdrroERVmS/3nSH0DyyzBU8="
    key_algorithm: "HMAC-SHA256"
    key_name: "public-openshift.example.com"
    server: "10.40.128.128"
```

**NOTE:** Use your values for `key_secret`, `key_name` and the server's IP
address.
