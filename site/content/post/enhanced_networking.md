---
date: 2018-09-22T20:04:40.407Z
title: T3 vs T2 ec2 instance comparison
---

## T2 vs T3

This blog does a simple and naive performance comparison to establish my opinionated view on
slowly gravitating towards Nitro(KVM) based hypervisors from XEN based hypervisors.
There are lot of videos on youtube describing about Nitro by AWS. There is excellent article by Brenden Gregg http://brendangregg.com/blog/2017-11-29/aws-ec2-virtualization-2017.html looking at the evolution of hypervisors used by AWS.

In this analysis am going to be using official CentOS7 AMI.
<img class="special-img-class" src="/images/enhanced_networking/CentOS7_1805_1.jpg" />

This has support for Cloud-init(0.7.9), which is good enought for bringing up instances
with basic config needed for our tests. 
There are 2 cloudformation scripts, one for bringing up
the networing(VPC & Subnets) and the other to bring up the instances used for testing. The scripts 
can be found at https://github.com/omshankar1/t2-t3-iperf3-tests

The vpc.yaml needs to be run first and the ec2 script would be using the VPC and Subnets created by vpc.yaml. At the time of writing, ap-southeast-2a does not have support for t3 instances. I had to use ap-southeast-2b for spinning up t3 instances.

First, lets spinup a pair of ec2 instances, a t2 and a t3 just for knowing the hypervisors details of each. The following screenshot captures the results of running ethtool on the interfaces.

<img class="special-img-class" src="/images/enhanced_networking/t2-t3-enasupport.jpg" />

Now, lets start the test running 2 t3.nano instances and comparing the network speed using iperf3. The test results are depicted below in the screenshot
<img class="special-img-class" src="/images/enhanced_networking/t3-nano-iperf3.jpg" />

Running the test with 2 t2.nano instances results in far lower network throughput, by about a magnitude less.
<img class="special-img-class" src="/images/enhanced_networking/t2-nano-iperf3.jpg" />

 Running the test with 2 m5.large instances results in about double the throughput observed in the case of t3.nano. m5.large also runs on Nitro based hypervisor.
<img class="special-img-class" src="/images/enhanced_networking/m5-large-iperf3.jpg" />

The results are on expected lines if we test speed between a t2 and an t3. This reminds of the proverb - a chain is only as strong as its weakest link :)

<img class="special-img-class" src="/images/enhanced_networking/t2-t3-iperf3-cmp.jpg" />

This is just tip of the iceberg. I believe we would also benefit on the IO throughput speed and speed to access S3 on moving from T2 to T3.