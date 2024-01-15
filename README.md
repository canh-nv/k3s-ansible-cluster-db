# Build a Kubernetes cluster using K3s via Ansible

Author: <https://github.com/itwars>  
Current Maintainer: <https://github.com/dereknola>

Easily bring up a cluster on machines running:

- [x] Debian
- [x] Ubuntu
- [x] Raspberry Pi OS
- [x] RHEL Family (CentOS, Redhat, Rocky Linux...)
- [x] SUSE Family (SLES, OpenSUSE Leap, Tumbleweed...)
- [x] ArchLinux

on processor architectures:

- [x] x64
- [x] arm64
- [x] armhf

## System requirements

The control node **must** have Ansible 8.0+ (ansible-core 2.15+)

All managed nodes in inventory must have:

- Passwordless SSH access
- Root access (or a user with equivalent permissions)

### Setup SSH access at the control node

- Follow config example bellow then add to `~/.ssh/config`
  ```
  Host 192.16.35.11
    HostName 192.16.35.11
    User canhnv
    IdentityFile ~/.ssh/canhnv
  ```

It is also recommended that all managed nodes disable firewalls and swap. See [K3s Requirements](https://docs.k3s.io/installation/requirements) for more information.

### Install Ansiable

- Use python to install ansible with latest versions
- Install python version that match with ansiable-core. Follow the matrix [here](https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix)
- Install ansiable with python [here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pip)

## Usage

First copy the sample inventory to `inventory.yml`.

```bash
cp inventory-sample.yml inventory.yml
```

Second edit the inventory file to match your cluster setup. For example:

```bash
k3s_cluster:
  children:
    server:
      hosts:
        192.16.35.11:
    agent:
      hosts:
        192.16.35.12:
        192.16.35.13:
```

If needed, you can also edit `vars` section at the bottom to match your environment.

If multiple hosts are in the server group the playbook will automatically setup k3s in HA mode with embedded etcd.
An odd number of server nodes is required (3,5,7). Read the [official documentation](https://docs.k3s.io/datastore/ha-embedded) for more information.

Setting up a loadbalancer or VIP beforehand to use as the API endpoint is possible but not covered here.

Start provisioning of the cluster using the following command:

```bash
ansible-playbook playbook/site.yml -i inventory.yml
```

Reset the nodes to state before installing k3s

```bash
ansible-playbook playbook/reset.yml -i inventory.yml
```

## Install Service Loadbalancer

- [Ingress-Nginx Controller](./ingress-nginx/README.md)

## Upgrading

A playbook is provided to upgrade K3s on all nodes in the cluster. To use it, update `k3s_version` with the desired version in `inventory.yml` and run:

```bash
ansible-playbook playbook/upgrade.yml -i inventory.yml
```

## Airgap Install

Airgap installation is supported via the `airgap_dir` variable. This variable should be set to the path of a directory containing the K3s binary and images. The release artifacts can be downloaded from the [K3s Releases](https://github.com/k3s-io/k3s/releases). You must download the appropriate images for you architecture (any of the compression formats will work).

An example folder for an x86_64 cluster:

```bash
$ ls ./playbook/my-airgap/
total 248M
-rwxr-xr-x 1 $USER $USER  58M Nov 14 11:28 k3s
-rw-r--r-- 1 $USER $USER 190M Nov 14 11:30 k3s-airgap-images-amd64.tar.gz

$ cat inventory.yml
...
airgap_dir: ./my-airgap # Paths are relative to the playbook directory
```

Additionally, if deploying on a OS with SELinux, you will also need to download the latest [k3s-selinux RPM](https://github.com/k3s-io/k3s-selinux/releases/latest) and place it in the airgap folder.

It is assumed that the control node has access to the internet. The playbook will automatically download the k3s install script on the control node, and then distribute all three artifacts to the managed nodes.

## Kubeconfig

After successful bringup, the kubeconfig of the cluster is copied to the control node and merged with `~/.kube/config` under the `k3s-ansible` context.
Assuming you have [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) installed, you can confirm access to your **Kubernetes** cluster with the following:

```bash
kubectl config use-context k3s-ansible
kubectl get nodes
```

If you wish for your kubeconfig to be copied elsewhere and not merged, you can set the `kubeconfig` variable in `inventory.yml` to the desired path.

## Local Testing

A Vagrantfile is provided that provision a 5 nodes cluster using Vagrant (LibVirt or Virtualbox as provider). To use it:

```bash
vagrant up
```

By default, each node is given 2 cores and 2GB of RAM and runs Ubuntu 20.04. You can customize these settings by editing the `Vagrantfile`.

## Need More Features?

This project is intended to provide a "vanilla" K3s install. If you need more features, such as:

- Private Registry
- Advanced Storage (Longhorn, Ceph, etc)
- External Database
- External Load Balancer or VIP
- Alternative CNIs

See these other projects:

- https://github.com/PyratLabs/ansible-role-k3s
- https://github.com/techno-tim/k3s-ansible
- https://github.com/jon-stumpf/k3s-ansible
- https://github.com/alexellis/k3sup
