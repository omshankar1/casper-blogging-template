---
date: 2018-09-22T20:04:40.407Z
title: Centos7 on a KVM and AWS using cloud-init
---

# Centos7 on a KVM and AWS using cloud-init


The intention is to bring up a Centos7 vm on KVM and AWS using cloudconfig.

The requirement is the KVM is installed already. I won't be talking about installation and the basics of KVM in this blog. This part is sufficielty documented in the internet. 
This blog is going to focus on a sample user-data which can be used and later modified to
provision a Centos7 instance.
The code for this blog can be found at github: https://github.com/omshankar1/Centos7_Cloudconfig

## Provision Centos7 on KVM

Spinning up a Centos7 vm involves couple of steps
- Creating the Subnets, in this case it would be linux bridges
- Use the virsh command to spinup the instance

### Creating linux bridges
The github repo has a script to create couple of bridges which will be used subsequently in provisioner script. 

Run the script virbr_network.sh to creates 3 linux bridges. 
Now list the interface using virsh net-list --all

```console
shankar@shankar-TP:~/KVM/github/Centos7_Cloudconfig$ virsh net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
 virbr1               active     yes           yes
 virbr2               active     yes           yes
 virbr3               active     yes           yes
```

We now have 3 linux bridges, essentially giving us the opportunity to create mulltiple interfaces. Next step will be to create a Centos7 machine with 2 interfaces
When we start fresh, there would be no virtual machine(vms) existing.

```console
shankar@shankar-TP:~/KVM/CENTOS/instance1$ virsh list --all
 Id    Name                           State
----------------------------------------------------
```

### Spinup Centos7 Instance

First step would be to write a user-data that uses a declaritive approach, for instance the configuration of yum packages, 
We can use virsh_provisioner.sh to create a Centos7 instance. When the instance comes we should be able to login without any password with user 'shankar' and ip address 192.168.1.14 specified in the user-data.
```yaml
#cloud-config
manage_resolv_conf: true
resolv_conf:
  nameservers: ['8.8.4.4', '8.8.8.8']

network: {config: disabled}
# Hostname management
preserve_hostname: False
hostname: centos7-test14
fqdn: centos7-test14

users:
  - name: "shankar"
    groups:
      - sudo
      - wheel
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCVR8MG7WnFVXoaPqyHApTbAC+x7MIt4ChaSihbuZi+CqkuuuszplsV2Px+is6cSrRbGB14uTg3XPPuL33YtKfvYjQTThiCiUz9F1FuUUKXhpNjRbvjfnF/zJFSmGJFOoUPN2gT2JzQfxtA29mFnklMK9iQyk43/X4sZkkRV/HoWPoA/If1+Q92rKB+miKqONwRuEdUOZxXwqva484/9SwvKtEUmemA4OLMVa8JMzbnkZbf+sqDK2GTTtaa3odOj6Z39cI7jf2Z8hrbdEdRZkB92qVINopI4eEqHvtdhGCgxOZ6Q6EAtzQsb2WPDqyJglN63UPy35pkxaXZaaxt/6DV shankar@shankar-TP"

packages:
  - yum-utils
  - wget
  - git
  - vim-enhanced
```


```bash
virt-install --os-type linux \
    --os-variant generic     \
    --name centos14     \
    --ram=1024     \
    --vcpus=1     \
    --cpu host     \
    --disk path=/home/shankar/KVM/CENTOS/instance1/centos.qcow2     \
    --network bridge=virbr1,mac=00:12:22:32:42:33     \
    --network bridge=virbr2,mac=00:13:23:33:43:33     \
    --cdrom nocloud.iso     \
    --console pty,target_type=serial     \
    --graphics none
```


```console
shankar@shankar-TP:~/KVM/CENTOS/instance1$ ssh shankar@192.168.1.14
Last login: Mon Sep 24 03:37:52 2018 from gateway
[shankar@centos7-test14 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:12:22:32:42:33 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.14/24 brd 192.168.1.255 scope global ens2
       valid_lft forever preferred_lft forever
    inet6 fe80::212:22ff:fe32:4233/64 scope link
       valid_lft forever preferred_lft forever
3: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:13:23:33:43:33 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.14/24 brd 192.168.2.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::213:23ff:fe33:4333/64 scope link
       valid_lft forever preferred_lft forever
```

If we list all the vms in KVM,

```console
shankar@shankar-TP:~/KVM/CENTOS/instance1$ virsh list --all
 Id    Name                           State
----------------------------------------------------
 5     centos14                       running
```

To view the cloud-init log

```console
[shankar@centos7-test14 ~]$ sudo tail -f /var/log/cloud-init.log
2018-09-24 03:58:00,612 - util.py[DEBUG]: Restoring selinux mode for /var/lib/cloud/instances/instance-1/sem/config_power_state_change (recursive=False)
2018-09-24 03:58:00,612 - helpers.py[DEBUG]: Running config-power-state-change using lock (<FileLock using file '/var/lib/cloud/instances/instance-1/sem/config_power_state_change'>)
2018-09-24 03:58:00,612 - cc_power_state_change.py[DEBUG]: no power_state provided. doing nothing
2018-09-24 03:58:00,612 - handlers.py[DEBUG]: finish: modules-final/config-power-state-change: SUCCESS: config-power-state-change ran successfully
2018-09-24 03:58:00,612 - main.py[DEBUG]: Ran 10 modules with 0 failures
2018-09-24 03:58:00,613 - util.py[DEBUG]: Creating symbolic link from '/run/cloud-init/result.json' => '../../var/lib/cloud/data/result.json'
2018-09-24 03:58:00,613 - util.py[DEBUG]: Reading from /proc/uptime (quiet=False)
2018-09-24 03:58:00,613 - util.py[DEBUG]: Read 14 bytes from /proc/uptime
2018-09-24 03:58:00,613 - util.py[DEBUG]: cloud-init mode 'modules' took 4.405 seconds (4.41)
2018-09-24 03:58:00,613 - handlers.py[DEBUG]: finish: modules-final: SUCCESS: running modules for final
```

### Provision Centos7 on AWS
We can use the exact same user-data and embed into CFN like in the script ec2_centos7_cloudconfig.yaml to provision centos7. I have commented out the parts that configures the ip interfaces.
I would need to indicate in the CFN the private  ip address for the first interface.
Also, I wouldn need to add an extra interface and attach it to the ec2 instance before I can run the complete user-data part

```yaml
      UserData: 
        Fn::Base64: !Sub |
          #cloud-config
          manage_resolv_conf: true
          resolv_conf:
            nameservers: ['8.8.4.4', '8.8.8.8']

          network: {config: disabled}
          # Hostname management
          preserve_hostname: False
          hostname: centos7-test14
          fqdn: centos7-test14

          users:
            - name: "shankar"
              groups:
                - sudo
                - wheel
              sudo: ['ALL=(ALL) NOPASSWD:ALL']
              ssh-authorized-keys:
                - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCVR8MG7WnFVXoaPqyHApTbAC+x7MIt4ChaSihbuZi+CqkuuuszplsV2Px+is6cSrRbGB14uTg3XPPuL33YtKfvYjQTThiCiUz9F1FuUUKXhpNjRbvjfnF/zJFSmGJFOoUPN2gT2JzQfxtA29mFnklMK9iQyk43/X4sZkkRV/HoWPoA/If1+Q92rKB+miKqONwRuEdUOZxXwqva484/9SwvKtEUmemA4OLMVa8JMzbnkZbf+sqDK2GTTtaa3odOj6Z39cI7jf2Z8hrbdEdRZkB92qVINopI4eEqHvtdhGCgxOZ6Q6EAtzQsb2WPDqyJglN63UPy35pkxaXZaaxt/6DV shankar@shankar-TP"

          packages:
            - yum-utils
            - wget
            - git
            - vim-enhanced
```