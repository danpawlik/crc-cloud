# Deploy CRC cloud directly on CRC host

## Introduction

The CRC cloud tool deploys the CRC host using its own binary, but it
requires access to the cloud provider credentials. That functionality
is useless when the CRC cloud needs to be used in externa CI, where
the crc-cloud tool can not be used - especially when the CI is responsible
to check how many VMs are spawned or checking job results.

## CRC QCOW2 image

There is a simply way to bootstrap the CRC without using the crc-cloud
tool and upload it to your cloud provider:

- download libvirt bundle (you can see url when crc is in debug mode)
- extract libvirt bundle using tar command with zst:

```shell
tar xaf crc_libvirt_$VERSION_amd64.crcbundle
```

- upload crc.qcow2 image to your cloud provider
- take the id_ecdsa_crc file

## Bootstrap crc-cloud directly on host

Now after spawning VM using the `crc.qcow2` image:

- prepare `inventory.yaml` file:

```shell
CRC_VM_IP=<ip address>

cat << EOF > inventory.yaml
---
all:
  hosts:
    crc-cloud.dev:
      ansible_port: 22
      ansible_host: $CRC_VM_IP
      ansible_user: core
  vars:
    openshift_pull_secret: |
      < PULL SECRET >
EOF
```

- clone crc-cloud project (FIXME: change repository)

```shell
git clone https://github.com/danpawlik/crc-cloud -b add-ansible-role
```

- run playbook to boostrap the container that later would start deploy-crc-cloud role

```shell
ansible-playbook -i inventory.yaml crc-cloud/ansible/playbooks/bootstrap.yaml
```
