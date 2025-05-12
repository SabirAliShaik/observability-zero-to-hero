## OpenTelemetry
- OpenTelemetry is an observability framework and toolkit designed to facilitiate the generation, Export, and Collection of telemetry data such as (traces, metrics, logs).
- OpenTelemetry is a Cloud Native Computing Foundation (CNCF) project that is the result of a merger between two prior projects, OpenTracing and OpenCensus. Both of these projects were created to solve the same problem: the lack of a standard for how to instrument code and send telemetry data to an Observability backend. As neither project was fully able to solve the problem independently, they merged to form OpenTelemetry and combine their strengths while offering a single solution.
- OpenTelemetry is Open source, as well as vendor- and tool-agnostic. it is not an observability backend itself.
  
## OpenTelemetry Architecture
- OpenTelemetry offers a unified standard for observability across multiple tools and vendors, unlike other libraries that may focus only on a specific aspect like tracing or metrics.
- 
![OpenTelemetry_arch](https://github.com/user-attachments/assets/f75132ff-5a77-4346-adfa-bcb6bcfd5ec8)


### 🖥️ Step 1: Create EKS Cluster

```bash
eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup
```
```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster observability \
    --approve
```
```bash
eksctl create nodegroup --cluster=observability \
                        --region=us-east-1 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking

# Update ./kube/config file
aws eks update-kubeconfig --name observability
```

### Step 2: Install OpenTelemetry otel-demo project using Helm

helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

helm install my-otel-demo open-telemetry/opentelemetry-demo

### Step 3: Verify the pods 

$kubectl get pods

Note: You need to wait for 5 mins till all pods are up and running

### Step 4: Access the application frontend service by port-forwarding

kubectl port-forward svc/forntend-proxy 8080:80 -n default

### Step 5: Add data in application
Access the application url : http://127.0.0.1:8080 and add items to cart and complete the order. This is to generate some data so that we can view the traces the Jaeger.

And also there is a fake load generator in project which generates the load for analysis.

### step 6: Verify request details in Jaeger & Grafana
This OTEL demo application comes with Jaeger, Grafana, Promethues installed. So, we should be able to access the same.

Grafana: http://127.0.0.1:8080/grafana/
Jaeger UI: http://127.0.0.1:8080/jaeger/ui/

We should be able to see dashboards created by this demo project by default in Grafana. 

## 🧼 Clean Up
```bash
helm uninstall my-otel-demo
eksctl delete cluster --name observability

```
