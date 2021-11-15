# PeerTube for Kubernetes
This PeerTube distribution is part of the [fedi-k8s](https://github.com/bobbyd0g/fedi-k8s) project, which aims to create a fully-featured social media hub that is easy to deploy to any modern cloud provider. This is a straightforward single-node deployment that can be achieved with kubectl Kustomize. Our hope is that this means anyone who's willing to learn can produce real community-controlled social resources for all their family and friends, and we can take back the Internet from stifling, compromised corporate platforms.

Made by [bobbyd0g](https://github.com/bobbyd0g) for the [Hellsite](https://hellsite.net), with big thanks to the [peertube-k8s](https://github.com/coopgo/peertube-k8s), [peertube-on-kubernetes](https://forge.extranet.logilab.fr/open-source/peertube-on-kubernetes), [peertube-helm](https://git.lecygnenoir.info/LecygneNoir/peertube-helm) and [peertubeexporter](https://gitlab.com/synacksynack/opsperator/docker-peertubeexporter/) projects for their important work.

Software used for this distribution includes:

[PeerTube](https://joinpeertube.org/) - [CrunchyData Postgres Operator v5](https://www.postgresql.org/about/news/pgo-the-crunchy-postgres-operator-v5-released-fully-declarative-postgres-2258/) - [Kubernetes NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/) - [cert-manager](https://cert-manager.io/) - [redis](https://hub.docker.com/_/redis/) - [Prometheus](https://prometheus.io/)

### Requirements

- You will need a working email address or delivery provider for this confiiguration, at least until the Postfix configuration is finished for fedi-k8s or on your own. You'll need some minimum of 64GB of block storage to do this right. S3 storage is also highly recommended.
- PeerTube is going to do some heavy lifting, so you probably want quad-core / 4GB RAM or better for this job. PeerTube does not support horizontal scaling behind a load balancer, so don't add replicas, upsize the node running your server.
- Install Kubernetes NGINX Ingress with TCP services configured, first to forward port 1935 for RTMP live streaming video. We can add more ports to the ConfigMap this generates later, or add UDP support with a fresh run of `helm upgrade`
```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set tcp.1935="hellsite-peertube/peertube:1935" \
  --set controller.metrics.enabled=true \
  --set-string controller.podAnnotations."prometheus\.io/scrape"="true" \
  --set-string controller.podAnnotations."prometheus\.io/port"="10254"
```
- Check the LoadBalancer this creates with `kubectl get svc -o wide` and enter its external IP address into an A record for your domain name.
- Install cert-manager and apply the [ClusterIssuer](https://cert-manager.io/docs/configuration/acme/) that will hand out your certificates to apps
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.6.0/cert-manager.yaml
```
- Install Crunchy PGO v5 
```
git clone https://github.com/CrunchyData/postgres-operator-examples.git
kubectl apply -k kustomize/install
```

### Deployment

Clone this repository into your working directory.
```
git clone https://github.com/bobbyd0g/k8s-tube.git
```
- Edit `peertube/kustomization.yaml` with configuration for your instance. The patches toward the bottom allow us to keep most of our edits neatly confined to this file. Choose a unique namespace and label, especially if planning to run multiple instances. Look for the `value` you want to change, usually the last line(s).
- Choose a nodeSelector that uniquely identifies the node you want this to run on. Your cloud provider will use some kind of `/metadata/labels/` that identifies the node pool it belongs to.
- Patch in the correct hostname for your instance, and the ClusterIssuer you set up for cert-manager. For Let's Encrypt, remember to use the staging server until you're absolutely certain you're done testing, because production is limited to 50 certs issued per month. You can also add any more ingress annotations here.
- Edit `pvc.yaml` with a storageClass that will `retain` detached volumes in case there's a problem.
- Edit `postgres-db/kustomization.yaml` with the same namespace, labels, and S3 creds. (If you want a trustless workflow, you know what you're doing, separate out the secrets :) )
- Edit `postgres.yaml` with the same `retain` -policy storageClass and nodeSelector (which is here broken out into `matchExpressions/key` and `matchExpressions/value`)
- Your backup schedule is configured here by default to retain a backup set (a full and its incrementals) for 22 days under `repo1-retention-full` -- and using cron formatting under `schedules` below, for a default full weekly backup on 1am Monday morning and daily incrementals at midnight.

- Create your namespace with `kubectl create namespace your-namespace`
- Bring up the database with `kubectl apply -k postgres-db` and give it a few minutes to become Ready.
- Bring up the website with `kubectl apply -k peertube` and look out for errors. Use `kubectl describe pod PODNAME` and `kubectl logs pod/PODNAME peertube` for troubleshooting.
- Get the root password from logs
```
kubectl get pod -n peertube
kubectl logs peertube-pod peertube | grep -A1 root
```

If you're having problems, you'll need to tear down both peertube with `kubectl delete -k peertube` and the database with `kubectl delete -k postgres-db` to do a clean re-initialization. The software used was chosen for particularly complete and accessible documentation. Beyond that, feel free to get in touch and we'll answer your questions as best we can. More docs and training material are forthcoming. Hope this helps!

# k8s-tube
