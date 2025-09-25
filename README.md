# Broadly Platform Infrastructure
**Author:** Daniel Maramba

## Objective

Orchestrate infrastructure for the Broadly platform. Deploy client and monitoring services.

## Prerequisites

### Helm

Helm must be installed and available via the CLI as `helm`.

### kubectl

Kubernetes in the command line must be installed and available as `kubectl`.

## Minikube

Start a minikube cluster:

```bash
minikube start -p broadly-staging
```

Start a minikube tunnel:

```bash
minikube tunnel -p broadly-staging
```

Set the context:

```bash
kubectl config use-context broadly-staging
kubectl config current-context
```

## Environment variables

Environment variables must be defined matching those expected by the overlying project.

Define client secrets:

```bash
kubectl -n default create secret generic client-secrets \
    --from-literal=NEXT_PUBLIC_APP_URL=VALUE # local development example: http://client.broadly (ensure there is a corresponding entry in the hosts file)
    --from-literal=NEXT_PUBLIC_ROOT_DOMAIN=VALUE # local development example: client.broadly (ensure there is a corresponding entry in the hosts file)
    --from-literal=NEXT_PUBLIC_ENABLE_SUBDOMAIN_ROUTING="false"
    --from-literal=DATABASE_URI="mongodb+srv://USERNAME:PASSWORD@HOST/DB"
    --from-literal=PAYLOAD_SECRET=VALUE # Found in the Payload dashboard
    --from-literal=STRIPE_SECRET_KEY=VALUE # Found in the Stripe dashboard
    --from-literal=STRIPE_ACCOUNT=VALUE  # Found in the Stripe dashboard
    --from-literal=STRIPE_WEBHOOK_SECRET=VALUE # Found in the Stripe dashboard
    --from-literal=BLOB_READ_WRITE_TOKEN=VALUE # Generated in the Vercel dashboard
```

Define monitoring secrets:

```bash
kubectl -n default create secret generic monitoring-secrets \
# DO WE NEED THIS?
    # --from-literal=INFLUX_DB_TOKEN=VALUE # Created in the Influx DB GUI
    --from-literal=INFLUX_DB_URL=VALUE # For example: http://localhost:8086/api/v2/write?org=broadly&bucket=jmeter
    --from-literal=INFLUX_TOKEN=VALUE # This can be generated before having to access the InfluxDB server. It should be a Base64URL. It is read on initialization by both services InfluxDB and Grafana
    --from-literal=GF_SECURITY_ADMIN_USER=VALUE # Grafana init admin user
    --from-literal=GF_SECURITY_ADMIN_PASSWORD=VALUE # Grafana init admin password
    --from-literal=DOCKER_INFLUXDB_INIT_USERNAME=VALUE # InfluxDB init admin password
    --from-literal=DOCKER_INFLUXDB_INIT_PASSWORD=VALUE # InfluxDB init admin password
```

## Initialize ingress controllers

From `/manifests/nginx`, run the following commands:

```bash
helm upgrade --install ingress-nginx-client ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --values client-values.yaml
```

```bash
helm upgrade --install ingress-nginx-influxdb ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --values influxdb-values.yaml
```

```bash
helm upgrade --install ingress-nginx-grafana ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --values grafana-values.yaml
```

```bash
helm upgrade --install ingress-nginx-prometheus ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --values prometheus-values.yaml
```

## Edit hosts file

1. Run the command:

    ``kubectl get svc -n ingress-nginx``

2. Find the IP under the column `EXTERNAL-IP`:

```bash
    NAME                                               ...   EXTERNAL-IP
    ingress-nginx-client-controller          ...   ##.###.###.###
    ...                                                      ...    <none>
    ...                                                      ...    <none>
```

3. Map the IP to the `alias` in /etc/hosts:

```bash
    sudo nano /etc/hosts
```

```bash
      /etc/hosts
      
      # add this line
      ##.###.###.###    client.broadly
```

4. Add similar lines for each `EXTERNAL-IP` returned by `kubectl get svc -n ingress-nginx`.

## Initialize ingress rules  

From `/ingress`, run `kubectl apply -f ./` 

## Initialize services

For each folder in `/services`, run `kubectl apply -f ./` 


## Cleanup

1. Delete secrets:

```bash
kubectl delete secret monitoring-secrets
kubectl delete secret client-secrets
```

2. Delete services

3. Uninstall ingress-nginx helm installations:

```bash
helm uninstall ingress-nginx-client -n ingress-nginx
helm uninstall ingress-nginx-grafana -n ingress-nginx
helm uninstall ingress-nginx-influxdb -n ingress-nginx
helm uninstall ingress-nginx-prometheus -n ingress-nginx
```

4. Power down minikube:

```bash
minikube stop -p broadly-staging
```
