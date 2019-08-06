---
layout: page
title: Quick Start
icon: fa-bolt
order: 2
---

**Practical Quick-Start Guide to create a simple 3 node cluster on OpenStack with LBaaS**

Please substitute all `%VARIABLES` with according values.

There are a few catches, such as image names, flavor names, network details. So double check them before trying this demo out.

Also, the certificate generated for the Kubernetes API is not signed for the Public IP it will get, so this is expected to throw an error for you if you use the default kubeconfig.
A workaround is to add the IP in your /etc/hosts as `kubernetes` and adjust your kubeconfig.
We're working on a better solution.

## Terraform example

```
variable "k8s" {
  default = {
    version = "1.14.3"
    image   = ""
    pubkeys = ["%YOUR_PUBLIC_KEY"]
    cni = {
      type    = "canal"
      version = "latest"
      extra   = false
    }
    storages = []
    etcd = {
      type      = "pod"
      discovery = "static"
      image     = ""
      nodes     = []
    }
    nodes = [
      {
        type    = "m1.small"
        image   = "%COREOS"
        labels  = ["master"]
        version = ""
      },
      {
        type    = "m1.small"
        image   = "%COREOS"
        labels  = ["compute"]
        version = ""
      },
      {
        type    = "m1.small"
        image   = "%COREOS"
        labels  = ["ingress"]
        version = ""
      },
    ]
    pki = {
      type = "local"
    }
    network = {
      cidr     = "192.168.123.0/24"
      base     = "50"
      dhcp     = true
      dns      = ["8.8.8.8"]
      upstream = "%EXTERNAL_NETWORK"
      fip      = true
      pool     = "%FLOATING_POOL"
    }
    loadbalancer = {
      enable = true
      type   = "lbaas"
    }
    ingress = {
      enable = true
      type   = "nginx-terranetes"
      specs  = {}
    }
  }
}

module "k8s" {
  source = "./core/kubernetes"
  k8s    = "${var.k8s}"
}

module "openstack" {
  source   = "./providers/openstack"
  k8s      = "${module.k8s.k8s}"
  ignition = "${module.k8s.ignition}"
  admin    = "${module.k8s.admin}"
}

output "k8s" {
  value = "${module.openstack.k8s}"
}

output "admin" {
  value = "${jsonencode(module.openstack.admin)}"
}
```