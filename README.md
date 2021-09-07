# Sock Shop : A Microservice Demo Application
### Enhanced for Gremlin CICD Pipeline Integrations

The appllication contained here is a fork of the `microservice demo application` found here:

https://github.com/microservices-demo

For more information on the base application, please refer to this repo.

## Overview

This repo contains installation instructions for enabling CICD Pipeline Integrations in this application as well as the Gremlin Client to enable attacks.

## Prerequisites

Ensure the following are installed on your system before continuing:

`kops`


## Creation of K8s cluster with Kops

To install the application to a `kops` cluster follow the following commands.

### Begin by creating a state store s3 bucket:

`aws s3api create-bucket --bucket cicd-demo-boosh-gremlin-rocks-state-store --region us-east-2 --create-bucket-configuration LocationConstraint=us-east-2`

### Enable versioning and encryption on this bucket:

`aws s3api put-bucket-versioning --bucket cicd-demo-boosh-gremlin-rocks-state-store  --versioning-configuration Status=Enabled`
`aws s3api put-bucket-encryption --bucket cicd-demo-boosh-gremlin-rocks-state-store --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'`

### For subsequent `kops` commands, a State Store is required, utilize the follwing environment variable for ease:

`export KOPS_STATE_STORE=s3://cicd-demo-boosh-gremlin-rocks-state-store`

### Create the `kops` cluster

`kops create cluster --name cicd-demo.boosh.gremlin.rocks --zones us-east-2a --master-size t3.medium --node-size t3.medium`

Where `name` is the desired name of your cluster, `zone` is your desired AWS availability zone, `master-size` and `node-size` are the EC2 machine sizes to be used for the master and worker nodes, respectively. For purposes of this demo, a `t3.medium` size is used. **NB: If no sizes are specified, `m4.large` is used by default. Please note this may incur larger than desired costs in AWS.**

This command will take some time to fully complete.

Upon completion, through `kops get cluster` should show output similar:

$ kops get cluster
NAME                            CLOUD   ZONES
cicd-demo.boosh.gremlin.rocks   aws     us-east-2a
$

To view the kops configuration that was created for editing and verification purposes, use the following:

`kops edit cluster --name cicd-demo.boosh.gremlin.rocks`

Upon validation of the config, this configuration must be applied:

`kops update cluster --name cicd-demo.boosh.gremlin.rocks --yes`

This will take 10-15 minutes and will run the actual creation of your k8s cluster.

Upon completion, output should be seen as similar to the following:

$ kops validate cluster --name cicd-demo.boosh.gremlin.rocks
Validating cluster cicd-demo.boosh.gremlin.rocks

INSTANCE GROUPS
NAME                    ROLE    MACHINETYPE     MIN     MAX     SUBNETS
master-us-east-2a       Master  t3.medium       1       1       us-east-2a
nodes-us-east-2a        Node    t3.medium       1       1       us-east-2a

NODE STATUS
NAME                                            ROLE    READY
ip-172-20-42-111.us-east-2.compute.internal     node    True
ip-172-20-55-245.us-east-2.compute.internal     master  True

Your cluster cicd-demo.boosh.gremlin.rocks is ready
$ 

## Deploy Sock Shop Microservices Application to K8s Cluster:

`kubectl create -f deploy/kubernetes/manifests/00-sock-shop-ns.yaml -f deploy/kubernetes/manifests`

## Deploy Gremlin to K8s Cluster

`https://www.gremlin.com/docs/infrastructure-layer/install-crio-containerd/`

`export GREMLIN_TEAM_ID=11111111-1111-1111-111111111111`
`export GREMLIN_TEAM_SECRET=1234567890-0987654321-1234567890`
`export GREMLIN_CLUSTER_ID=my-cluster`

Export gremlin variables

`kubectl create namespace gremlin`
`helm repo add gremlin https://helm.gremlin.com/`
`helm install gremlin gremlin/gremlin --namespace gremlin --set gremlin.hostPID=true --set gremlin.container.driver=containerd-runc --set gremlin.secret.managed=true --set gremlin.secret.teamID=$GREMLIN_TEAM_ID --set gremlin.secret.clusterID=$GREMLIN_CLUSTER_ID --set gremlin.secret.teamSecret=$GREMLIN_TEAM_SECRET --set gremlin.secret.type=secret`

## Enable security groups in AWS to expose web application