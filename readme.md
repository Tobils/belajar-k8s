# Kubernetes

Belajar kubernetes dari programmer zaman now

## Install

- install minikube
- install kubectl

## Node

- bisa dalam bentuk fisik atau vm
- setiap node memiliki kube-proxy, kubelet
- setiap node terdiri dr berbagai pod
- commands

  ```bash
  # melihat nodes
  kubectl get nodes
  # output
  NAME       STATUS   ROLES                  AGE   VERSION
  minikube   Ready    control-plane,master   9d    v1.20.2

  # melihat detail node
  kubectl describe node namanode
  # contoh
  kubectl describe node minikube
  ```

## Pod

- nama pod unique
- unit terkecil dr k8s cluster
- 1 pod bisa running beberapa container
- pod itu aplikasi yang running dalam k8s
- commands

  ```bash
  # melihat pod
  kubectl get pods

  # melihat detail pod
  kubectl describe pod namapod
  ```

- create config yml pod:example nginx.yml
  ```yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
      - name: nginx
        image: nginx
        ports:
          - containerPort: 80
  ```
- commands

  ```bash
  # membuat pod
  kubectl create -f nameconfigfile.yml
  # contoh
  kubectl create -f examples/nginx.yml
  # outpout
  pod/nginx created

  # get list pod
  kubectl get pods -o wide
  NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
  nginx   1/1     Running   0          96s   172.18.0.5   minikube   <none>           <none>

  # akses pod
  kubectl port-forward namapod portAkses:porPod
  # contoh
  kubectl port-forward nginx 8080:80
  # output
  Forwarding from 127.0.0.1:8080 -> 80
  Forwarding from [::1]:8080 -> 80
  Handling connection for 8080
  Handling connection for 8080
  ```

## Label

- kenapa butuh label ?
  - untuk memberi tanda pod
  - untuk mengorganisir pod
  - memberi informasi tambahan pod
- label bisa digunakan untuk semua resources k8s tidak hanya untuk pod
- config file

  ```yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-with-label
    labels:
      team: finance
      version: 1.4.5
      environment: production
  spec:
    containers:
      - name: nginx
        image: nginx
        ports:
          - containerPort: 80
  ```

- comands

  ```bash
  # create pod
  kubectl create -f examples/nginx-with-label.yml

  # show pod with label
  kubectl get pods --show-labels
  # output
  NAME               READY   STATUS    RESTARTS   AGE   LABELS
  nginx              1/1     Running   0          13m   <none>
  nginx-with-label   1/1     Running   0          28s   environment=production,team=finance,version=1.4.5

  # memberi label pod yang sudah running
  kubectl label pod nginx environtment=development
  # output
  NAME               READY   STATUS    RESTARTS   AGE     LABELS
  nginx              1/1     Running   0          15m     environtment=development
  nginx-with-label   1/1     Running   0          2m39s   environment=production,team=finance,version=1.4.5

  # overwrite or update label
  kubectl label pod nginx environtment=qa --overwrite
  # output
  NAME               READY   STATUS    RESTARTS   AGE     LABELS
  nginx              1/1     Running   0          16m     environtment=qa
  nginx-with-label   1/1     Running   0          3m47s   environment=production,team=finance,version=1.4.5

  # query pod dengan label
  kubectl get pods -l key
  kubectl get pods -l key=value
  kubectl get pods -l key!=value
  kubectl get pods -l !key
  kubectl get pods -l key1,key2=value


  # contoh
  kubectl get pods -l environment
  kubectl get pods -l 'environment in (qa, production)'
  ```
