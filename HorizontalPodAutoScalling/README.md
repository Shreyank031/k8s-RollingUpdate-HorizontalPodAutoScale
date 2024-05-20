## PHP-Apache Deployment with Horizontal Pod Autoscaler

This YAML manifest defines a Kubernetes Deployment, Service, and HorizontalPodAutoscaler (HPA) for a PHP application running on an Apache web server.

## Deployment

The Deployment named php-apache is responsible for managing the Pods running the PHP-Apache application.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 500m
          limits:
            cpu: 1000m
```

- The Deployment is configured to maintain 1 replica of the PHP-Apache application.
- The Pods are selected using the label `app=php-apache`.
- The container image used is `k8s.gcr.io/hpa-example`.
- The container exposes port 80 for the web server.
- The Deployment requests 500 millicore (0.5 CPU) and limits the CPU usage to 1000 millicore (1 CPU) for each Pod.

## Service

The Service named php-apache is used to expose the `PHP-Apache` application to other components within the cluster or outside the cluster (if configured accordingly).

> 1vCPU = 1000milicore. It's a absolute unit.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    app: php-apache
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: php-apache
```
- The Service is configured to listen on port 80.
- It forwards traffic to the Pods' port 80 (`targetPort: 80`).
- The Service selects Pods with the label `app=php-apache`.

## HorizontalPodAutoscaler

The HorizontalPodAutoscaler (HPA) named php-apache is responsible for automatically scaling the Deployment based on the CPU utilization of the Pods.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

- The HPA targets the php-apache Deployment.
- It sets the minimum number of replicas to 1 and the maximum number of replicas to 10.
- The autoscaling is based on the CPU utilization metric.
- The target CPU utilization is set to 50%.

- The `minReplicas` field sets the minimum number of replicas that the HPA will maintain for the target resource (Deployment). In this case, it is set to 1, meaning that there will always be at least one replica (Pod) running.

- The `maxReplicas` field sets the maximum number of replicas that the HPA can scale up to. In this case, it is set to 10, meaning that the HPA can scale the target Deployment up to a maximum of 10 replicas (Pods).

- The `metrics` field specifies the metric(s) that the HPA will use to determine when to scale the target resource. In this case, there is only one metric defined, which is a `Resource` type metric.

- The resource field under `metrics` specifies the `resource` to be monitored, which is `cpu` in this case.

- The `target` field under `resource` defines the desired target value for the `metric`. In this case, the type is set to `Utilization`, and the `averageUtilization` is set to 50.

- This means that the HPA will monitor the average CPU utilization across all Pods of the target Deployment. If the average CPU utilization exceeds 50%, the HPA will scale up the Deployment by adding more replicas (up to the maximum of 10). If the average CPU utilization drops below 50%, the HPA will scale down the Deployment by removing replicas (down to the minimum of 1).

- Avg cpu utilization exceeds 50% means the 50% of request cpu that we have mentioned in our deployment. That's equal to `50% of 500milicore(request cpu) = 250milicore`. If it exceeds the `250milicore` then a new replica of pod is created even though we have set the replica to be 1 in our deployment manifest. 

- It continues to create replicas of the Pod until the maximum of 10 replicas is reached (as specified in the maxReplicas field of the HPA). If the incoming traffic is too high, and Kubernetes cannot handle it even with 10 replicas, then one or more of the Pods might reach 100% CPU utilization, but Kubernetes will not terminate them. Instead, it will try to schedule additional Pods, but if there are no available resources in the cluster, those Pods will remain in a pending state."

## Usage

1. Prerequisites

- A running Kubernetes cluster. Minikube is fine
- kubectl configured to interact with your cluster

2. Clone the repository:

```
git clone https://github.com/Shreyank031/k8s-RollingUpdate-HorizontalPodAutoScale.git
cd k8s-RollingUpdate-HorizontalPodAutoScale/HorizontalPodAutoScalling
```

3. Aplly the YAML manifest. I have bundled the Service, Deploymenta and HPA under single Manifest

```
kubectl apply -f hpa-php-apache.yaml
```

4. Verify if everything is working as expected.
```
kubectl get all 
```

5. Create the load.
```

# Run this in a separate terminal
# so that the load generation continues and you can carry on with the rest of the steps

kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

6. Watch how the pods are scalled one by one with respect to load. 
```
#Execute all 3 commands in seperate terminal. Use tmux.

kubectl get hpa --watch
kubectl get rs --watch
kubectl get pods --watch

```

7. Finally `delete` the `deployment`, `svc` and `hpa` to see the pods going back to 0 after spike.
```
kubectl delete deploy php-apache
kubectl delete svc php-apache
kubectl delete hpa php-apache
```


