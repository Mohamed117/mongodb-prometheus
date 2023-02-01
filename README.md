# mongodb-prometheus
Expose MongoDB to Prometheus in Kubernetes cluster

## Setting Up Prometheus + Grafana 

 `helm repo add prometheus-community https://prometheus-community.github.io/helm-charts `<br> 
 `helm repo add stable https://charts.helm.sh/stable ` <br>
 `helm repo update `<br>
 `helm install prometheus prometheus-community/kube-prometheus-stack `<br>

## Access Grafana UI
* Exposing **prometheus-grafana** deployment by a LoadBalancer service: <br>
`kubectl expose deployment prometheus-grafana  --type=LoadBalancer --name=exposed-grifana`

* Extracting the Login credentials: <br>
``kubectl get secret prometheus-grafana -o yaml
`` <br> <br>
get the _admin-password_ and _admin-user_ values from the output above and decode them [here](https://www.base64decode.org/) <br> <br>
default values: <br>
  admin-user: admin <br>
  admin-password: prom-operator
  
* You can access Grafana's dashboard using the external IP address of the newly created **exposed-grifana** service


## Access Prometheus UI
* Exposing **prometheus-kube-prometheus-prometheus** ClusterIP service by a LoadBalancer service: <br>
`kubectl expose service prometheus-kube-prometheus-prometheus  --type=LoadBalancer --name=exposed-prometheus`
  
* You can access Prometheus's dashboard using the external IP address of the newly created **exposed-prometheus** service


## Deploy MongoDB application (Deployment and Service component)

* Apply the mongodb.yml file <br>
`kubectl apply -f mongodb.yml`


## MongoDB Exporter - exposing MongoDB metrics

* Export helm chart values to a yaml file and edit it <br>
`helm show values prometheus-community/prometheus-mongodb-exporter > values.yml` <br>
`vim values.yml`

* Edit the values as the following:
> mongodb: <br>
  >  &nbsp; &nbsp; uri: "mongodb://mongodb-service:27017"
<br>

> serviceMonitor: <br>
> &nbsp; &nbsp; interval: 90s <br>
> &nbsp; &nbsp; scarpeTimeout 60s <br>
> &nbsp; &nbsp; enabled: true <br>
>  &nbsp; &nbsp; additionalLabels: <br>
>   &nbsp; &nbsp; &nbsp; &nbsp; release: prometheus

* Install it using the new values file: <br>
`helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f values.yml`

## Check the mongoDB metric exporter
* Exposing **mongodb-exporter-prometheus-mongodb-exporter** ClusterIP service by a LoadBalancer service: <br> 
`kubectl expose service mongodb-exporter-prometheus-mongodb-exporter  --type=LoadBalancer --name=exposed-mongodb-exporter` <br>
* You can access it using the external IP address of the newly created **exposed-mongodb-exporter** service








