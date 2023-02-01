# mongodb-prometheus
Expose MongoDB to Prometheus in Kubernetes cluster

## Setting Up Prometheus + Grifana 
```
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo add stable https://charts.helm.sh/stable 
$ helm repo update 
$ helm install prometheus prometheus-community/kube-prometheus-stack
```
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
  
* You can access Grifana's dashboard using the external IP address of the newly created **exposed-grifana** service


## Access Prometheus UI
* Exposing **prometheus-kube-prometheus-prometheus** ClusterIP service by a LoadBalancer service: <br>
`kubectl expose service prometheus-kube-prometheus-prometheus  --type=LoadBalancer --name=exposed-prometheus`
  
* You can access Prometheus's dashboard using the external IP address of the newly created **exposed-prometheus** service


## Deploy MongoDB application (Deployment and Service component)

* Apply the mongodb.yml file <br>
`kubectl apply -f mongodb.yml`


## MongoDB Exporter - exposing MongoDB metrics

















