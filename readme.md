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

## Annotations

- untuk apa ?
  - mirip label tapi tidak dapat difilter
  - untuk menambahkan informasi tambahan | deskripsi | string json
  - bisa menampung sampai 256 kb
- command

  ```bash
  # running pod
  kubectl create -f examples/nginx-with-annotation.yml

  # check details
  kubectl describe pod nginx-with-annotation
  #output
  Name:         nginx-with-annotation
  Namespace:    default
  Priority:     0
  Node:         minikube/172.17.0.2
  Start Time:   Sun, 06 Mar 2022 09:56:08 +0700
  Labels:       environment=development
                team=product
                version=1.1.1
  Annotations:  apapun: apapun itu ........................
                description: annotasi deskripsi dari sebuah value yang didefinisikan, apapun itu bisa dimmasukan kesini
  ...

  # menambahkan annotation ke dalam pod
  kubectl annotate pod namepod key=value
  kubectl annotate pod namepod key=value --overwrite
  ```

## Nammespace [Grouping]

- pod dgn nama yg sama boleh berjalan di namespace yg berbeda
- pod bukanlah cara untuk mengisolasi resources
- walaupun berbeda namespace, pod akan tetap bisa saling berkomunikasi dengan pods yg lain.
- command

  ```bash
  # create namespace
  kubectl create -f examples/finance-namespace.yml

  # crate pods with name space
  kubectl create -f examples/nginx.yml --namespace finance

  # get pods with nammespace
  kubectl get pod --namespace finance

  # cara mmenghapus namespace => hati2 saat naespace dihapus resources di dalammnya akan terhapus [pod2 akan dterhapus]
  kubectl delete namespace nama-namespace
  ```

## Mengahapus POD

- command

  ```bash
  kubectl delete pod nama-pod
  kubectl delete pod nama-pod1 nama-pod2 nama-pod3

  # mengahpus pod mmenggunakan label
  kubectl delete pod -l key=value

  # mengahapus semua pod di Namespace
  kubectl delete pod --all --namespace nama-namespace
  ```

## Probe

- pengecekan di k8s
- next materi
  - replication controller dan replica set
- harus mmengerti terlebih dahulu tentang liveness, readiness, dan startup probe di k8s
- Liveness, Readiness, Startup
  - pengecekan dilakukan oleh kubelet
  - liveness
    - pengecekan secara periodic kapan perlu mme-restart pod
  - readiness
    - pengecekan secara periodic, jika pod tidak sehat maka trafic ke dalam pod akan dihentikan sampai kebali sehat
  - startup
    - pengecekan setatus kesehatan diawal untuk memastikan aplikasi kita siap untuk berjalan
- mekanisme pengecekan Probe
  - HTTP Get (cocok untuk aplikasi web)
  - TCP Socket
  - Commmand Exec
- konfigurasi pprobe
  - initialDelaySeconds jgn sampai 0 second, aplikasi butuh startup
  - periodSeconds, seberapa sering pengecekan dilakukan, default 10
  - timeoutSeconds, waktu pengecekan ketika pengecekan gagal, default 1
  - successThreshold, minimum dianggap sukses setelah berstatus failure, default 1
  - failureThreshold, minimum dianggap gagal, default 3

## Replication Controller

- memastikan pod selalu berjalan
- jika tiba2 pod mati atau hilang misal ketika ada node yg mati, maka replication controller secara otomatis akan menjalankan pod yg bati atau yg hilang tsb.
- biasanya ditugaskan untuk memanage lebih dari 1 pod
- memastikan jumlah pod yg berjalan sesuai dengan yg ditugaskan, jika kurang maka akan menambah pod yang baru dan jika lebih maka akan mengurangi pod yg ada.
- isi replication controller
  - Label selector sebagai penanda pod
  - replica count, jumlah pod yang harus berjalan
  - pod template, template yang digunakan untuk menjalankan pod
- command

  ```bash
  kubectl create -f examples/nginx-rc.yml

  # melihat replication controller
  kubectl get replicationcontrollers
  kubectl get replicationcontrollers
  kubectl get rc

  # menghapus rc by default akan menghapus pod
  kubectl delete rc rcname
  # tanpa menghapus pod
  kubectl delete rc rcname --cascade=false
  ```

## Replica Set

- awal mula replica controller digunakan untuk menjaga jumlah pod
- sekarang dikenalkan yang baru bernama replica set
- replica set merupakan egenrasi baru dari replica controller dan digunakan sebagai pengganti replica controller
- penggunaan replication controller sekrang ini sudah tidak direkomendasikan
- replica set vs replica controller
  - replica set memiliki kemampuan hampir sama dengan replica controller
  - replica set memiliki label selector yg lebih expressive
  - replica controller hanya memiliki fitur selector secara match [key=value]
- match expression => selector menggunakan expresi
  - In, value label harus ada di value in
  - NotIn, value label tidak boleh ada di value in
  - Exists, label harus ada
  - NotExists, label tidak boleh ada

## Upgrade minikube

- command

  ```bash
  minikube update-check
  CurrentVersion: v1.20.0
  LatestVersion: v1.25.2

  # download mminikube lnatest release : https://github.com/kubernetes/minikube/releases
  sudo cp minikube-linux-amd-64 /usr/local/bin
  sudo chmod +x minikube-linux-amd-64
  sudo rm -rf minikube
  sudo mv minikube-linux-amd-64 minikube

  # check new upgraded
  minikube update-check
  CurrentVersion: v1.25.2
  LatestVersion: v1.25.2

  minikube start

  # running dgn virtual box
  minikube start --vm-driver=virtualbox

  # running dgn virtual box dan config cpu
  minikube start --vm-driver=virtualbox --cpus=2 --memory=2g --disk-size=20g

  ```
