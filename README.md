# Kubernetes Federated Cluster Playbook

The following is an attempt to automate the creation of a federated Kubernetes cluster (on Google Cloud) since current methods are either manual, have issues running on Mac or buggy.

Manifests are taken from http://kubernetes.io/docs/admin/federation/ with much reference to Kelsey Hightower's guide at https://github.com/kelseyhightower/kubernetes-cluster-federation

> NOTE: this is currently work in progress

## Requirements

- Ansible 2.1+
- Google Cloud SDK installed
- Google Cloud account
- Google Cloud Project created
- Google Cloud DNS enabled
- Kubernetes CLI tools

## Running the Playbook

- First modify the `site.yml` file with your specific environment settings
- Run `ansible-playbook site.yml`

## Useful Commands & Information

after the playbook finishes successful run the following and ensure all clusters have a `Ready` service**
`kubectl --context=federation-cluster get clusters -w`

to test the cluster, you can create an nginx service
`kubectl --context=federation-cluster create -f test/nginx-service.yaml`
you should see one `Ingress` address per cluster.

sometimes it's useful to delete the cluster registration
`kubectl --context=federation-cluster delete cluster <cluster_name1> <cluster_name2>`

the playbook makes a reasonable attempt at being idempotent so you should be able to keep running the playbook to fix any issues to occur
