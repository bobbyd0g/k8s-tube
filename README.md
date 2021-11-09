## Easy PeerTube for Kubernetes.
This PeerTube distribution is the first of a number of components for the fedi-k8s project, which endeavors to present a well-documented turnkey high-availability Fediverse portal solution for easy deployment and maintenance. It can also be deployed standalone on a cluster already configured with Ingress-NGINX, cert-manager, a mailserver, and Crunchy Postgres Operator.

Made by [bobbyd0g](https://github.com/bobbyd0g) for the [Hellsite](https://hellsite.net), with big thanks to the [peertube-k8s](https://github.com/coopgo/peertube-k8s), [peertube-on-kubernetes](https://forge.extranet.logilab.fr/open-source/peertube-on-kubernetes), and [peertube-helm](https://git.lecygnenoir.info/LecygneNoir/peertube-helm) projects for laying the groundwork.

Software used for this distribution includes:

[Peertube](https://joinpeertube.org/) <br />
[Kustomize](https://kustomize.io/) <br />
[CrunchyData Postgres Operator v5](https://www.postgresql.org/about/news/pgo-the-crunchy-postgres-operator-v5-released-fully-declarative-postgres-2258/)<br />
[cert-manager](https://cert-manager.io/)<br />
[redis](https://hub.docker.com/_/redis/)<br />
[NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)<br />

### Advantages
- Using Crunchy PGO v5, we have a fully-declarative Postgres solution spinning up properly-configured high-availability databases, with replication, backup, monitoring, and more.
- With cert-manager, we can manage certs alongside any modern ingress controller with high availability that can use Kubernetes Ingress resources.
- NGINX Ingress Controller is relatively easy to use, forgiving of mistakes, and allows our full configuration with a single Ingress resource.
- Kustomize allows us to release manifests that are centrally configurable without destructively editing values strewn throughout a bunch of .example files.

### Where to Improve
- RTMP live video streaming isn't working yet because Ingress-NGINX TCP services are giving me a headache.
- There are enough variables to warrant writing a Helm chart.
- Needs thorough testing and more investigation into behaviors in HA deployment.

## Deployment

- First, install your ingress controller, cert-manager, and Crunchy Postgres Operator v5 or later. Documentation is forthcoming.
- Check each of the manifests for options that apply to you, look for the comments, but change as little as possible at first.
- Enter your variables into kustomization.yaml
- Install the Crunchy database first with apply -k postgres-db
- Apply the deployment configuration with apply-k peertube
- Do not increase replicas until db initialization is complete
- More detail to come in short order
# k8s-tube
