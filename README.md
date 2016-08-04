# Kubernetes Federated Cluster Playbook

The following is an attempt to automate the creation of a federated Kubernetes cluster (on Google Cloud) since current methods are either manual, have issues running on Mac or buggy.

Manifests are taken from http://kubernetes.io/docs/admin/federation/ with much reference to Kelsey Hightower's guide at https://github.com/kelseyhightower/kubernetes-cluster-federation

## Requirements

- Ansible 2.1+
- Google Cloud SDK installed
- Google Cloud account
- Google Cloud Project created
- Google Cloud DNS enabled
- Kubernetes CLI tools (`kubectl` version 1.3+)

## Running the Playbook

- First modify the `site.yml` file with your specific environment settings
- Run `ansible-playbook site.yml`

## Useful Commands & Information

- After the playbook finishes successfully, run the following and ensure all clusters have a `Ready` service.

`kubectl --context=federation-cluster get clusters -w`

- To test the cluster, you can create an nginx service.

`kubectl --context=federation-cluster create -f test/nginx-service.yaml`
you should see one `Ingress` address per cluster.

- Since for now Service is the only federated service, that is, that is automatically distributed to all clusters, you need to create deployments, pods etc. manually on each cluster.

```
# list all contexts
for c in $(kubectl config view -o jsonpath='{.contexts[*].name}'); do echo $c; done

# for each of the cluster contexts
kubectl --context="<cluster_name>" \
  run nginx --image=nginx:1.11.1-alpine --port=80
```

- Sometimes it's useful to delete the cluster registration.

`kubectl --context=federation-cluster delete cluster <cluster_name1> <cluster_name2>`

- The playbook makes a reasonable attempt at being idempotent so you should be able to keep running the playbook to fix any issues that occur.

- Note the `DNS_NAME` should be a real DNS name and the dot at the end is important. However, it does not need to be resolvable to get the Federated Cluster to come up.

- If everything is working as it should, you should see A records created in the specified DNS zone:

`gcloud dns record-sets list -z federation`

- Cleanup can be done with the following:
```
kubectl --context=federation-cluster delete services nginx
kubectl --namespace=federation delete pods,svc,rc,deployment,secret --all
gcloud dns managed-zones delete federation

# and for each cluster
gcloud container clusters delete <cluster_name> --zone=<cluster_zone>
```

- This is an early attempt at automating a Federated Cluster and since there is ongoing work to improve the current method, this playbook may need to change significantly in the future, I will however attempt to keep improving it while it is still useful.

- Pull Requests welcome

- Thanks to madhusudancs@google.com for providing valuable assistance getting this working, who is also happy to assist with Kubernetes Cluster Federation related topics.
