---
date: 2018-09-22:24:16-04:00
title: Centos7 on a AWS using cloud-init
---


In the last blog we saw how to create a Centos7 instance on KVM. In this blog we'll have a look at ways to create a sample config files to bring up a Centos7 vm on AWS using the exact same cloud-init. 

We'll create a VPC and subnets. As we plan to create an ec2 instance with two interfaces, with each on a different Subnet, they need to be within same AZ. In this case we'll be using only SubnetPubA and SubnetPvtA.

There are two Cloudformation scripts in th github repository vpc.yaml and ec2.yaml. The vpc.yaml which needs to be run first creates the necessary network and exports the vpc and subnet ids. These exported values will be imported in ec2.yaml.

vpc.yaml
```yaml
Outputs:
  VPCId:
    Description: VPCId
    Value: !Ref EnvironmentVPC
    Export:
      Name: !Sub VPC:Id
  SubnetPubAId:
    Description: SubnetPubAId
    Value: !Ref SubnetPubA
    Export:
      Name: !Sub SubnetPubA:Id
  SubnetPvtAId:
    Description: SubnetvtAId
    Value: !Ref SubnetPvtA
    Export:
      Name: !Sub SubnetPvtA:Id
```

ec2.yaml
```yaml
  DataInterface:
    Type: AWS::EC2::NetworkInterface
    Properties:
      Tags:
        - Key: Name
          Value: DataInterface
      PrivateIpAddress: 10.1.2.100
      SubnetId: !ImportValue 'SubnetPvtA:Id'

```

This snippet of cloudformation ensures that both SubnetPubA and SubnetPvtA are within the same AZ regardless of the region.
```yaml
  SubnetPubB:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref EnvironmentVPC
      CidrBlock: !Ref SubnetPubBCidr
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select 
        - 1
        - Fn::GetAZs: !Ref 'AWS::Region'

  SubnetPvtA:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref EnvironmentVPC
      CidrBlock: !Ref SubnetPvtACidr
      MapPublicIpOnLaunch: false
      AvailabilityZone: !Select 
        - 0
        - Fn::GetAZs: !Ref 'AWS::Region'
```
## Provision Centos7 on AWS
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
                - "ssh-rsa AAAAB3Nz..."

          packages:
            - yum-utils
            - wget
            - git
            - vim-enhanced
```