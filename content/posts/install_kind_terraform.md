+++
title = 'Setting up kind cluster with ingress and helm using terraform'
date = 2025-01-06
draft = false
+++

There are multiple options to have kubernetes installed locally for local development or testing.
Some of this options are:
- <a href="https://minikube.sigs.k8s.io/docs/" target="_blank">Minikube</a>
- <a href="https://k3d.io/v5.6.3/" target="_blank">k3d</a>
- <a href="https://docs.docker.com/desktop/setup/install/mac-install/" target="_blank">Docker for mac</a>
- <a href="https://kind.sigs.k8s.io/" target="_blank">kind</a>

Today I will setup a local Kubernetes cluster using terraform.

kind is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI as well.

Configuration files used

***# versions.tf***
```
terraform {
  required_providers {
    kind = {
      source  = "tehcyx/kind"
      version = "0.2.0"
    }

    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "2.22.0"
    }

    helm = {
      source  = "hashicorp/helm"
      version = "2.10.1"
    }

    null = {
      source  = "hashicorp/null"
      version = "3.2.1"
    }
  }
```

***# variables.tf***
```
variable "kind_cluster_name" {
  type        = string
  description = "The name of the cluster."
  default     = "demo-local"
}

variable "kind_cluster_config_path" {
  type        = string
  description = "The location where this cluster's kubeconfig will be saved to."
  default     = "~/.kube/config"
}

variable "ingress_nginx_helm_version" {
  type        = string
  description = "The Helm version for the nginx ingress controller."
  default     = "4.7.1"
}

variable "ingress_nginx_namespace" {
  type        = string
  description = "The nginx ingress namespace (it will be created if needed)."
  default     = "ingress-nginx"
}
```

***# kind_cluster.tf***
```
provider "kind" {
}

provider "kubernetes" {
  config_path = pathexpand(var.kind_cluster_config_path)
}

resource "kind_cluster" "default" {
  name            = var.kind_cluster_name
  kubeconfig_path = pathexpand(var.kind_cluster_config_path)
  wait_for_ready  = true

  kind_config {
    kind        = "Cluster"
    api_version = "kind.x-k8s.io/v1alpha4"

    node {
      role = "control-plane"

      kubeadm_config_patches = [
        "kind: InitConfiguration\nnodeRegistration:\n  kubeletExtraArgs:\n    node-labels: \"ingress-ready=true\"\n"
      ]
      extra_port_mappings {
        container_port = 80
        host_port      = 80
      }
      extra_port_mappings {
        container_port = 443
        host_port      = 443
      }
    }

    node {
      role = "worker"
    }
  }
}
```
Since we are using **~/.kube/config** for the variable **kind_cluster_config_path** value, pathexpand is used to take a filesystem path that might begin with a ~ segment, and replace that segment with the current user's home directory path.
If **pathexpand** would not be used it will literally write it out to **~/.kube/config** within the current working directory.

***# nginx_ingress.tf***
```
provider "helm" {
  kubernetes {
    config_path = pathexpand(var.kind_cluster_config_path)
  }
}

resource "helm_release" "ingress_nginx" {
  name       = "ingress-nginx"
  repository = "https://kubernetes.github.io/ingress-nginx"
  chart      = "ingress-nginx"
  version    = var.ingress_nginx_helm_version

  namespace        = var.ingress_nginx_namespace
  create_namespace = true

  values = [file("nginx_ingress_values.yaml")]

  depends_on = [kind_cluster.default]
}

resource "null_resource" "wait_for_ingress_nginx" {
  triggers = {
    key = uuid()
  }

  provisioner "local-exec" {
    command = <<EOF
      printf "\nWaiting for the nginx ingress controller...\n"
      kubectl wait --namespace ${helm_release.ingress_nginx.namespace} \
        --for=condition=ready pod \
        --selector=app.kubernetes.io/component=controller \
        --timeout=90s
    EOF
  }

  depends_on = [helm_release.ingress_nginx]
}
```
This **values = [file("nginx_ingress_values.yaml")]** line was added because if you want nginx ingress to work with kind a bit of extra configuration is needed.
The **null_resource** waits that pods with the label **app.kubernetes.io/component=controller** in the nginx ingress are running, and it time's out after 90 seconds.

***# nginx_ingress_values.yaml***
```
controller:
  hostPort:
    enabled: true
  terminationGracePeriodSeconds: 0
  service:
    type: "NodePort"
  watchIngressWithoutClass: true
  nodeSelector:
    ingress-ready: "true"
  tolerations:
  - effect: "NoSchedule"
    key: "node-role.kubernetes.io/master"
    operator: "Equal"
  - effect: "NoSchedule"
    key: "node-role.kubernetes.io/control-plane"
    operator: "Equal"
  publishService:
    enabled: false
  extraArgs:
    publish-status-address: "localhost"
```
<br>

**For those who use Colima on Mac**
<br>

You need to first stop colima ***colima stop*** and then start it again using ***colima start --network-address*** and then ***sudo ssh -F ~/.colima/ssh_config colima -L 80:172.18.0.3:80 -L 443:172.18.0.3:443***


**Usage**
Run **terraform apply**
Deploy a test app to verify that ingress works kubectl **apply -f https://kind.sigs.k8s.io/examples/ingress/usage.yaml**
