# PeerTube for Kubernetes
This PeerTube distribution is the first of a number of components for the [fedi-k8s](https://github.com/bobbyd0g/fedi-k8s) project, which endeavors to create a well-documented turnkey Fediverse portal solution for easy deployment and maintenance. It can also be deployed standalone on a cluster that is already configured with the prerequisite software.

Made by [bobbyd0g](https://github.com/bobbyd0g) for the [Hellsite](https://hellsite.net), with big thanks to the [peertube-k8s](https://github.com/coopgo/peertube-k8s), [peertube-on-kubernetes](https://forge.extranet.logilab.fr/open-source/peertube-on-kubernetes), and [peertube-helm](https://git.lecygnenoir.info/LecygneNoir/peertube-helm) projects for their important work.

Software used for this distribution includes:

[Peertube](https://joinpeertube.org/) <br />
[CrunchyData Postgres Operator v5](https://www.postgresql.org/about/news/pgo-the-crunchy-postgres-operator-v5-released-fully-declarative-postgres-2258/)<br />
[Kubernetes NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)<br />
[cert-manager](https://cert-manager.io/)<br />
[redis](https://hub.docker.com/_/redis/)<br />

## Features

- Single-node PeerTube instance for Kubernetes, bound with nodeSelector
- Automatically-maintained database on the same node for optimal performance
- Easy to deploy in your typical cloud environment with block and object storage
- PeerTube has direct access to the rest of your cluster for integration purposes
- All major features are supported, including RTMP Live Streaming and live replay
- Striving for a 100% declarative YAML workflow, with a few surmountable exceptions

### Why build this stack?

- Crunchy PGO v5 is a fully-declarative solution to automate the creation and maintenance of highly-available replicated PostgreSQL clusters, with scheduled backup to S3
- cert-manager automates the issuance and maintenance of SSL certificates across all of your domains and applications, all using the same Kubernetes Ingress resources as...
- Kubernetes NGINX Ingress Controller is fast, reliable, powerful, and easy to configure.

### Requirements

All requirements will be addressed by the fedi-k8s distribution:
- Kubernetes NGINX Ingress Controller installed (with TCP support for live streaming)
- CrunchyData Postgres Operator v5 installed to your cluster
- cert-manager configured with your issuer and interface
- Access to an SMTP server and account for outbound mail
- It can be disabled, but we highly recommend using S3 object storage for videos.
- This configuration will consume two volumes using your default StorageClass.

### Limitations

- PeerTube does not support horizontal scaling behind a load balancer. Don't increase replicas, or many things will break, even including configuration.
- Livestreaming capacity is extremely limited -- how to integrate external infrastructure?
- Still needs Prometheus monitoring. Outbound email will be provided centrally in fedi-k8s

## Deployment

This section will be worked out in more detail very shortly.
- First, edit postgres.yaml to configure S3 storage, check the deployment.yaml PVC.
- Add your values to the kustomization.yaml file; PGO fills the db details from secret
- Install ingress-nginx with TCP services support. Connect external load balancer
```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  -f values.yaml
```
```
tcp:
  1935: "peertube/peertube:1935"
```
- Install cert-manager with default configuration, and apply your ClusterIssuer
- Install Crunchy PGO v5 with default configuration
- Create your namespace (default is "peertube")
- Provision the database using Kustomize
```
kubectl apply -k postgres-db
```
- Apply your TLS secret to the website namespace if you have one already.
- Give the database a minute to initialize, then bring up the website.
```
kubectl apply -k peertube
```
- Get the root password from logs
```
kubectl get pod -n peertube
kubectl logs peertube-pod peertube | grep -A1 root
```
Log in, create your first user, and get posting!

# k8s-tube