# ansible-role-rockylinux-ami-builder

## Overview
This Ansible role provides a fully automated way to create a custom Rocky Linux
AMI for the AWS Cloud environment. Unlike most other AMI creation programs this
role does not derive an existing AMI where additional packages and configuration
files would be added. The AMI is created from scratch by partitioning a disk and
installing the operating system using packages from the official Rocky Linux
repositories. This allows to have a custom disk layout with LVM (Logical Volume
Manager) and to control all packages which are going to be installed in order to
get a really minimal system.

## Features
This Ansible role can produces Rocky Linux v9 AMI images for both x86_64
and arm64 architectures in AWS. These images can be used with modern instance
types such as t3/t3a and t4g. On x86_64 it boots the system in Legacy BIOS mode,
and on arm64 it boots in UEFI mode, as these are the default boot modes with
AWS for these two architectures. The AMI is configured to have both IPv4 and
IPv6 enabled, and it is possible to connect to the instance using IPv6 only,
as you may want to run instances without an EIP, as AWS have introduced relatively
high charges to attach public IPv4 external IP addresses on EC2 instnaces.
The disk layout is based on LVM as it is very useful for allocating the disk
space and it can be quite important on stateful instances such as database
servers to make sure there is still space for data even if logs are growing a
lot. The image will come with cloud-init in order to configure the SSH key which
is selected at launch time. Also it will automatically grow the LVM
Physical-Volume so all the disk space in the EBS volume of the instance can be
allocated to grow the root file system or create other file systems with LVM.

## Requirements
This Ansible role comes with the following requirements:
   * An AWS Account where to execute the creation of the AMI
   * An access key pair with privileges to use EC2 and Cloudformation
   * Ansible 2.10.0 (or more recent) with the module for AWS and Boto3
   * Permissions to launch the official Rocky Linux v9 AMIs for your architecture

## How it works
This Ansible role works by first using Cloud Formations to create a temporary
stack in your AWS Account which contains a VPC and an EC2 instance. This
instance is running an official Rocky Linux AMI and the new Rocky Linux
operating system will be installed on a secondary empty EBS volume. At the end
of the installation the instance is stopped and the new AMI is created as an
image of the secondary volume. The stack and all its resources are destroyed
and only the new AMI and its corresponding EBS snapshot remain.

## How to use it
   * Make sure you have a compatible version of Ansible with the module for AWS
     as well as the Boto3 python library
   * Install this role using ansible-galaxy
     https://galaxy.ansible.com/fdupoux/rockylinux_ami_builder
   * Subscribe to the following official Rocky Linux AMIs in the AWS marketplace:
     - https://aws.amazon.com/marketplace/pp/prodview-6ihwigagrts66
     - https://aws.amazon.com/marketplace/pp/prodview-ygp66mwgbl2ii
   * Use this role from an ansible playbook and make sure you pass all the
     important parameters which are defined in the defaults/main.yml file.
     Please have a look at the example playbook provided in the docs folder
     for examples of parameters that can be used for x86_64 and arm64.

## Ansible
This role has been tested using Ansible 2.10.7 as provided with Alpine Linux
v3.14 but it should also work with more recent versions of Ansible. A dockerfile
has been provided so you can run a container which has this version of Ansible
which is known to work with this role and which has all dependencies required
(ansible and python modules). You will have to mount volumes to give the container
access to your SSH Key, your AWS Access Key pair, the ansible role and playbook.

## Testing
The creation of an AMI for x86_64 has been tested using a "t3.small" instance,
and an AMI for arm64 has been created using a "t4g.small" instance. These two
instance types come with modern features and are affordable. This should also
work on other modern instance types, but may not work with older instance
types such as t1 and t2.
