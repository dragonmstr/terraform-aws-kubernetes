# AWS Kubernetes

AWS Kubernetes is a Kubernetes cluster deployed using [Kubeadm](https://kubernetes.io/docs/admin/kubeadm/) tool. It provides full integration with AWS. It is able to handle ELB load balancers, EBS disks, Route53 domains etc.

<!-- TOC -->

- [AWS Kubernetes](#aws-kubernetes)
    - [Updates](#updates)
    - [Prerequisites and dependencies](#prerequisites-and-dependencies)
    - [Including the module](#including-the-module)
    - [Addons](#addons)
    - [Custom addons](#custom-addons)
    - [Tagging](#tagging)

<!-- /TOC -->

## Updates

* *28.9.2017:* Split into module and configuration
* *22.8.2017:* Update Kubernetes and Kubeadm to 1.7.4
* *30.8.2017:* New addon - Fluentd + ElasticSearch + Kibana
* *2.9.2017:* Update Kubernetes and Kubeadm to 1.7.5

## Prerequisites and dependencies

* AWS Kubernetes deployes into existing VPC / public subnet. If you don't have your VPC / subnet yet, you can use [this](https://github.com/scholzj/terraform-aws-vpc) module to create one.
  * The VPC / subnet should be properly linked with Internet Gateway (IGW) and should have DNS and DHCP enabled.
  * Hosted DNS zone configured in Route53 (in case the zone is private you have to use IP address to copy kubeconfig and access the cluster).
* To deploy AWS Kubernetes there are no other dependencies apart from [Terraform](https://www.terraform.io). Kubeadm is used only on the EC2 hosts and doesn't have to be installed locally.

## Including the module

Although it can be run on its own, the main value is that it can be included into another Terraform configuration which is using Kubeadm.

```hcl
module "kubernetes" {
  source = "github.com/scholzj/terraform-aws-minikube"

   aws_region    = "eu-central-1"
  cluster_name  = "aws-kubernetes"
  master_instance_type = "t2.medium"
  worker_instance_type = "t2.medium"
  ssh_public_key = "~/.ssh/id_rsa.pub"
  ssh_access_cidr = ["0.0.0.0/0"]
  api_access_cidr = ["0.0.0.0/0"]
  min_worker_count = 3
  max_worker_count = 6
  ...
  ...
  ...
}
```

An example of how to include this can be found in the [examples](examples/) dir.

## Addons

Currently, following addons are supported:
* Kubernetes dashboard
* Heapster for resource monitoring
* Storage class for automatic provisioning of persisitent volumes
* External DNS (Replaces Route53 mapper)
* Ingress
* Autoscaler
* Logging with Fluentd + ElasticSearch + Kibana

The addons will be installed automatically based on the Terraform variables. 

## Custom addons

Custom addons can be added if needed. For every URL in the `addons` list, the initialization scripts will automatically call `kubectl -f apply <Addon URL>` to deploy it. The cluster is using RBAC. So the custom addons have to be *RBAC ready*.

## Tagging

If you need to tag resources created by your Kubernetes cluster (EBS volumes, ELB load balancers etc.) check [this AWS Lambda function which can do the tagging](https://github.com/scholzj/aws-kubernetes-tagging-lambda).
