## Simple GO REST API:

This project contains a simple GOLang rest API service. This service is useful to test the Kubernetes and Docker.

We also packaged it as a Docker Image. We used mutli stage build to create the small docker image. and **The size of the image is very small. It's just 12.4 MB.**
You can build/push/pull it from docker hub using below command.
```
$ docker build -t davarski/go-rest-api-demo .
$ docker login
$ docker push davarski/go-rest-api-demo
$ docker pull davarski/go-rest-api-demo
```
## API Details:
We are using the Gorilla Mux and Following routes are implemented

- `/info` - Returns the info about the request.
- `/health-check` - Returns JSON response as `True` if the service is running fine. We can also add custom delay by setting the `HEALTHDELAY` Environment variable.
- `/env` - Returns present container environment variables in JSON format.
- `/quote` - Returns a Random Quote from the `quotes.json` file.


## How to Run:

Clone the repo and run the `main.go` using the `go run`

```
$ go run main.go 
2022/08/01 12:05:54 Simple API Server(1.0.0) running on 0.0.0.0:32000
```

By default server listens on `10000` Port on `all Interfaces (0.0.0.0)` of node. Goto the `127.0.0.1:10000/info` from your browser or by using the `curl`.

## Running using the Docker:
Please look at the [Running on Docker](https://github.com/adavarski/REST-go-k8s-example#running-on-docker) section

## Running using Kubernetes(K8s):
Please look at the [Running on Kubernetes](https://github.com/adavarski/REST-go-k8s-example#running-on-kubernetes) section

### `/info` Endpoint output:
```
{"Endpoint":"/info","Host":"127.0.0.1:10000","Method":"GET","RemoteIP":"127.0.0.1:60942","Version":"1.0.0"}
```

### Curl Example:
```
$ curl 127.0.0.1:10000/info
{"Endpoint":"/info","Host":"127.0.0.1:10000","Method":"GET","RemoteIP":"127.0.0.1:60942","Version":"1.0.0"}
```

### `/health-check` endpoint:
This `/health-check` route, Returns `healthy:true` if server is running fine. So we can see the status of the server by monitoring this route. This can be used with the Kubernetes Readiness and Liveness probes.
We can also add custom delay by setting the `HEALTHDELAY` environment variable.

```
$ curl 127.0.0.1:10000/health-check
{"healthy":true}
```

### `/env` endpoint:

This `/env` route dumps the all environment variables

```
$ curl 127.0.0.1:10000/env
{"Version":"1.0.0","env":"CLUTTER_IM_MODULE=xim, LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.
...


```

### `/quote` endpoint:

This `/quote` route will generate random quote. We use the `quotes.json` file, which contains the list of quotes with author names. You can pass your own `json` quotes file to override the quotes.

```
$ curl 127.0.0.1:10000/quote
{"Quote":"If my mind can conceive it, if my heart can believe it, then I can achieve it. - Muhammad Ali","Version":"1.0.0"}
$ curl 127.0.0.1:10000/quote
{"Quote":"Any fool can write code that a computer can understand. Good programmers write code that humans can understand. - Martin Fowler","Version":"1.0.0"}
$ curl 127.0.0.1:10000/quote
{"Quote":"The only true wisdom is in knowing you know nothing. - Socrates","Version":"1.0.0"}


```

## Changing the Port Number:

We can change the Port number where API server is running by setting the `PORT` environment variable.

By default server runs on `10000` Port.

```
$ go run main.go 
2022/08/01 12:14:18 Simple API Server(1.0.0) running on 0.0.0.0:10000

```

Set the `PORT` Environment variable and run the `go run`. If you are in Linux set using the `export` command.

```
$ export PORT="32000"
$ go run main.go 
2022/08/01 12:18:19 Simple API Server(1.0.0) running on 0.0.0.0:32000

```

We can see the `PORT` is changed.


## Health-check Delayed response:

We can delay the `/health-check` endpoint response by setting the  **`HEALTHDELAY`** Environment variable.

```
$ export HEALTHDELAY="5"
```

Re-run the app
```
$ go run main.go 
2022/08/01 12:21:59 Simple API Server(1.0.0) running on 0.0.0.0:10000
2022/08/01 12:22:06 Entering /health-check endpoint, Invoked from 127.0.0.1:36984
2022/08/01 12:22:11 Added 5 seconds delay, Serving Requst Now

$ curl 127.0.0.1:10000/health-check
{"healthy":true}

```

> Note that the response is served after 5 seconds at `00:43:53`

## Passing your own Quotes(`quotes.json`) file:

You can change the source Quotes fine by setting the `QUOTESFILE` environment variable. Please look at the following example.

```
$ cp quotes.json test.json
$ export QUOTESFILE="./test.json" 
$ go run main.go 
2022/08/01 12:36:38 Simple API Server(1.0.0) running on 0.0.0.0:10000
2022/08/01 12:36:50 Entered /quote endpoint, Invoked from 127.0.0.1:45226 , using ./test.json as QuotesSource
2022/08/01 12:36:50 The only true wisdom is in knowing you know nothing. - Socrates

$ curl 127.0.0.1:10000/quote
{"Quote":"The only true wisdom is in knowing you know nothing. - Socrates","Version":"1.0.0"}


$ unset QUOTESFILE 
$ go run main.go 
2022/08/01 12:39:15 Simple API Server(1.0.0) running on 0.0.0.0:10000
2022/08/01 12:39:20 Entered /quote endpoint, Invoked from 127.0.0.1:46610 , using ./quotes.json as QuotesSource
2022/08/01 12:39:20 If my mind can conceive it, if my heart can believe it, then I can achieve it. - Muhammad Ali

$ curl 127.0.0.1:10000/quote
{"Quote":"If my mind can conceive it, if my heart can believe it, then I can achieve it. - Muhammad Ali","Version":"1.0.0"}


```
> Here I have added a `test.json` file on the same folder, So I have used relative Path. Please make sure to specify the correct path for file (or use the Absolute path)


# Running on Docker:

You can create the docker container using the `docker run` like below.

```
$ docker run -d -p 10000:10000 davarski/go-rest-api-demo
26b56788f3cf4521e64d19fbe1a6cd480e4a52012a0130b05e928f86b86654da

```
By default server listens on `10000` Port on `all Interfaces (0.0.0.0)` of node. We have exposed the port `10000` to the host system. So that I can test with `curl` or any other app.

## Changing the runtime behavior on Docker:

### Changing the Port Number:

We can change the Port number where API server is running by setting the `PORT` environment variable. By default server runs on `10000` Port.

Here is an example to change the default port number using the docker environment variable. Use the `-e` option and pass the desired port number to `PORT` environment variable.

```
$ docker run -d -e PORT='32000' -p 32000:32000 davarski/go-rest-api-demo
b9dbfdda77dfd365ae6ff71a8117c8d29669e81620af281efdcec227976f142a
```

You can check the logs to see where the server is listening.
```
$ docker logs 51b5ce01e53f
2022/08/01 09:44:53 Simple API Server(1.0.0) running on 0.0.0.0:32000

```
Here it is listening on `0.0.0.0:32000` 

Make a call to any of the API endpoint or route.
```
$ curl 127.0.0.1:32000/quote
{"Quote":"The only way to learn a new programming language is by writing programs in it. - Dennis Ritchie","Version":"1.0.0"}


```

### Health-check Delayed response:

We can delay the `/health-check` endpoint response by setting the  **`HEALTHDELAY`** Environment variable. The configurable health-check API is useful to test the `Readiness and Liveness Probes of K8s`

Here I am setting the `5 Seconds` delay by passing the `5` to `HEALTHDELAY` env.

```
$ docker run -d -e HEALTHDELAY='5' -p 10000:10000 davarski/go-rest-api-demo
bcbcd1e06093c82e5a8a62ee01df8fa4f0f6048f8e0c535d726e9b8385c07b78

```

Now try to access the `/health-check` API. and you will get the delayed response ( You are response delayed by 5 seconds).

```
$ curl 127.0.0.1:10000/health-check
{"healthy":true}

```

We can also verify the delay by looking into the docker container logs. 

```
$ docker logs bcbcd1e06093
2022/08/01 09:47:43 Simple API Server(1.0.0) running on 0.0.0.0:10000
2022/08/01 09:48:18 Entering /health-check endpoint, Invoked from 10.30.0.1:48810
2022/08/01 09:48:23 Added 5 seconds delay, Serving Requst Now
 
```
We can see the request was received at ` 18:40:08` and we delayed it by 5 seconds and served the request at ` 18:40:13`.


## K8s deploy (KIND):

```
## Install docker, kubectl, etc.

## Instal KIND

$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64 && chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind

## Create cluster (CNI=Calico, Enable ingress)

$ kind create cluster --name devops --config cluster-config.yaml

$ kind get kubeconfig --name="devops" > admin.conf
$ export KUBECONFIG=./admin.conf 

$ kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
$ kubectl -n kube-system set env daemonset/calico-node FELIX_IGNORELOOSERPF=true


## Ingress Nginx
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

## LoadBalancer (MetalLB)

$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml

### Edit metallb-configmap-davar.yaml
```
$ docker network inspect -f '{{.IPAM.Config}}' kind
[{172.20.0.0/16  172.17.0.1 map[]} {fc00:f853:ccd:e793::/64   map[]}]

$ cat metallb-configmap-carbon.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.20.0.200-172.20.0.250
```

## Deploy Go microservcie:


create a pod definition file and specify the container image as `mvenkatesh431/k8s-test-container`. Here is an example.

```
$ cat kubernetes/loadbalancer-usage.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rest
  labels:
    app: rest
spec:
  replicas: 2
  selector:
    matchLabels:
      app: rest
  template:
    metadata:
      labels:
        app: rest
    spec:
      containers:
      - name: rest
        image: davarski/go-rest-api-demo
        ports:
        - name: http
          containerPort: 10000
---
kind: Service
apiVersion: v1
metadata:
  name: rest
spec:
  type: LoadBalancer
  selector:
    app: rest
  ports:
  # Default port used by the image
  - port: 10000

```

Let's create the Go microservcie using the `kubectl create` command, You can also use the `kubectl apply` command,

```
$ kubectl apply -f kubernetes/loadbalancer-qotd.yaml 
deployment.apps/rest created
service/rest created
davar@carbon:~/Documents/REST-gRPC-Go-k8s-playground/REST-go-k8s-example$ kubectl get all
NAME                       READY   STATUS              RESTARTS   AGE
pod/rest-786fc49b6-ks57w   0/1     ContainerCreating   0          6s
pod/rest-786fc49b6-p7hdv   0/1     ContainerCreating   0          6s

NAME                 TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)           AGE
service/kubernetes   ClusterIP      10.96.0.1    <none>         443/TCP           86m
service/rest         LoadBalancer   10.96.9.7    172.20.0.200   10000:32595/TCP   6s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rest   0/2     2            0           6s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/rest-786fc49b6   2         2         0       6s
davar@carbon:~/Documents/REST-gRPC-Go-k8s-playground/REST-go-k8s-example$ kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/rest-786fc49b6-ks57w   1/1     Running   0          13s
pod/rest-786fc49b6-p7hdv   1/1     Running   0          13s

NAME                 TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)           AGE
service/kubernetes   ClusterIP      10.96.0.1    <none>         443/TCP           86m
service/rest         LoadBalancer   10.96.9.7    172.20.0.200   10000:32595/TCP   13s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rest   2/2     2            2           13s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/rest-786fc49b6   2         2         2       13s

```

let's check if the pod is created using the kubectl get pods
```
$ kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/rest-786fc49b6-ks57w   1/1     Running   0          13s
pod/rest-786fc49b6-p7hdv   1/1     Running   0          13s

NAME                 TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)           AGE
service/kubernetes   ClusterIP      10.96.0.1    <none>         443/TCP           86m
service/rest         LoadBalancer   10.96.9.7    172.20.0.200   10000:32595/TCP   13s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rest   2/2     2            2           13s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/rest-786fc49b6   2         2         2       13s

$ curl http://172.20.0.200:10000/info
{"Endpoint":"/info","Host":"172.20.0.200:10000","Method":"GET","RemoteIP":"192.168.188.128:36300","Version":"1.0.0"}
$ curl http://172.20.0.200:10000/health-check
{"healthy":true}
$ curl http://172.20.0.200:10000/env
{"Version":"1.0.0","env":"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin, HOSTNAME=rest-786fc49b6-ks57w, KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1, REST_SERVICE_PORT=10000, REST_PORT=tcp://10.96.9.7:10000, REST_PORT_10000_TCP=tcp://10.96.9.7:10000, REST_PORT_10000_TCP_PROTO=tcp, REST_PORT_10000_TCP_ADDR=10.96.9.7, KUBERNETES_SERVICE_HOST=10.96.0.1, KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443, REST_SERVICE_HOST=10.96.9.7, KUBERNETES_SERVICE_PORT=443, KUBERNETES_SERVICE_PORT_HTTPS=443, KUBERNETES_PORT=tcp://10.96.0.1:443, KUBERNETES_PORT_443_TCP_PORT=443, REST_PORT_10000_TCP_PORT=10000, KUBERNETES_PORT_443_TCP_PROTO=tcp, HOME=/root, VERSION=1.0.0"}
$ curl http://172.20.0.200:10000/quote
{"Quote":"There is no great genius without some touch of madness. - Aristotle","Version":"1.0.0"}
$ curl http://172.20.0.200:10000/quote
{"Quote":"The only true wisdom is in knowing you know nothing. - Socrates","Version":"1.0.0"}
$ curl http://172.20.0.200:10000/quote
{"Quote":"Programming isn't about what you know; it's about what you can figure out. - Chris Pine","Version":"1.0.0"}

```

Now the pod is created. You can also check the pod logs using the `kubectl log` command like below.
```
$ kubectl get po
NAME                   READY   STATUS    RESTARTS   AGE
rest-786fc49b6-ks57w   1/1     Running   0          13m
rest-786fc49b6-p7hdv   1/1     Running   0          13m
$ kubectl logs rest-786fc49b6-ks57w
2022/08/01 10:02:06 Simple API Server(1.0.0) running on 0.0.0.0:10000
2022/08/01 10:03:02 Entered /quote endpoint, Invoked from 192.168.188.128:34597 , using ./quotes.json as QuotesSource
2022/08/01 10:03:02 The only way to learn a new programming language is by writing programs in it. - Dennis Ritchie
2022/08/01 10:03:38 Entering /env endpoint, Invoked from 192.168.188.128:30288
2022/08/01 10:03:43 Entered /quote endpoint, Invoked from 192.168.188.128:27348 , using ./quotes.json as QuotesSource
2022/08/01 10:03:43 There is no great genius without some touch of madness. - Aristotle
2022/08/01 10:03:45 Entered /quote endpoint, Invoked from 192.168.188.128:21150 , using ./quotes.json as QuotesSource
2022/08/01 10:03:45 The only true wisdom is in knowing you know nothing. - Socrates
2022/08/01 10:03:47 Entered /quote endpoint, Invoked from 192.168.188.128:59392 , using ./quotes.json as QuotesSource
2022/08/01 10:03:47 Programming isn't about what you know; it's about what you can figure out. - Chris Pine

```

Get more details about the pod using the `kubectl describe pod` command.
```
$ kubectl describe po rest-786fc49b6-ks57w
Name:         rest-786fc49b6-ks57w
Namespace:    default
Priority:     0
Node:         devops-worker/172.20.0.2
Start Time:   Mon, 01 Aug 2022 13:01:57 +0300
Labels:       app=rest
              pod-template-hash=786fc49b6
Annotations:  cni.projectcalico.org/containerID: 18b60815475ccfd0364f2df981f940f503c3ba7fd0ce1c0f746d816c27bd2e53
              cni.projectcalico.org/podIP: 192.168.188.136/32
              cni.projectcalico.org/podIPs: 192.168.188.136/32
Status:       Running
IP:           192.168.188.136
IPs:
  IP:           192.168.188.136
Controlled By:  ReplicaSet/rest-786fc49b6
Containers:
  rest:
    Container ID:   containerd://2f46ecba197c19c2812e21235f15821984f7ebd451c4ab122f6b536e23876a9d
    Image:          davarski/go-rest-api-demo
    Image ID:       docker.io/davarski/go-rest-api-demo@sha256:4d632eacd6ddbc6143e7aaea760adb8e167788e272dc636dbce54b371401fd8a
    Port:           10000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 01 Aug 2022 13:02:06 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n5vf7 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-n5vf7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  14m   default-scheduler  Successfully assigned default/rest-786fc49b6-ks57w to devops-worker
  Normal  Pulling    14m   kubelet            Pulling image "davarski/go-rest-api-demo"
  Normal  Pulled     13m   kubelet            Successfully pulled image "davarski/go-rest-api-demo" in 7.37499621s
  Normal  Created    13m   kubelet            Created container rest
  Normal  Started    13m   kubelet            Started container rest

```

