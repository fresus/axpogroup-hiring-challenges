
# SRE Instrumentation Challenge - Candidate: Jesús A. Rodríguez

## Step 1: Implementation

* I'm using the python package [prometheus-flask-exporter](https://github.com/rycus86/prometheus_flask_exporter) 
  to expose the http metrics to prometheus because is easy and convenient.
* The docker image is midly optimized to work with python but it could be interesting to:
  * Add a WSGI server (e.g. gunicorn)
  * Add a user to run the app instead of using root
  * Add pip cache mounted from the host to avoid installing the same packages on every docker build
  * Use a package manager (eg. uv, poetry) to save the exact packages version in a lock file, among other benefits
* In the docker compose file it could be interesting to use a different user than root to run the containers,
  as it was mentioned in the previous point


## Step 2: Visualization

* Steps to reproduce the result of this part:
  * Start docker compose:
  ```shell
  $ docker compose start -d
  ```

  * Run the script to generate requests:
  ```shell
  $ scripts/generate_traffic.sh
  ```

  * Go to [Grafana](http://localhost:3000) > Dashboards > New dashboard

* The issue causing the 500 code errors was that the `delete_bucket` function ended up returning a 500 code instead
  of the correct one, 200. It should be fixed now.


## Step 3: Deployment

Used a minikube cluster with ingress and a [local registry](https://minikube.sigs.k8s.io/docs/handbook/registry/) 
enabled to test the deployment:
```shell
$ minikube start --addons=registry,ingress
$ docker run --name="registry-proxy" -d --rm -it --network=host alpine ash -c "apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TCP:$(minikube ip):5000"
```

### Storage API

* Modified the service name to `storage-api` because `storage_api` is not compliant with the resource names 
  that kubernetes allows
* Built and pushed the docker image to the local registry with:
```shell
$ docker build -t localhost:5000/storage-api:latest ./src
$ docker push localhost:5000/storage-api
```

* Generated the kubernetes manifest with these commands and modified them accordingly when needed:
  ```shell
  $ kubectl create deployment storage-api \
    --image=localhost:5000/storage-api:latest \
    --port=5000 \
    --dry-run=client \
    -o yaml > deploy/kubernetes/storage-api/deployment.yaml
  $ kubectl create service clusterip storate-api \
    --tcp=5000:5000 \
    --dry-run=client \
    -o yaml > deploy/kubernetes/storage-api/service.yaml
  $ kubectl create ingress storage-api \
    --rule="storage-api.example.local/"=storage-api:5000 \
    --dry-run=client \ 
    -o yaml > deploy/kubernetes/storage-api/ingress.yaml
  ```

* Created the kubernetes resources with:
```shell
$ kubectl apply -f deploy/kubernetes/storage-api
```

* Added a new line in `/etc/hosts` to resolve the storage-api domain:
```shell
$ echo "$(minikube ip) storage-api.example.local" | sudo tee -a /etc/hosts
```

* Modified the `generate_traffic.sh` script to allow different domains, now it's possible to use it
  like this:
```shell
$ scripts/generate_traffic.sh "storage-api.example.local"
```

### Prometheus

* Generated the kubernetes manifests in the same way as in the previous point but also added
  the prometheus configuration (including it accordingly in the deployment manifest):
```shell
$ kubectl create configmap prometheus-config \
  --from-file=deploy/prometheus/prometheus.yml \
  --dry-run=client \
  -o yaml > deploy/kubernetes/configmap.yaml
```

* Modified the prometheus configuration to reflect the service name change from `storage_api` to `storage-api`

