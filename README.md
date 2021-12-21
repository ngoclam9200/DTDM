# KUBERNETES AWS(EC2)

## CHUẨN BỊ
Xây dựng hai máy ubuntu trên EC2.
|Role| 
|----|
|Master|
|Worker|

![](https://github.com/ngoclam9200/DTDM/blob/master/File/anh%20readme/2mayao.png)
## Install Kubernetes Cluster using kubeadm
Hướng dẫn thiết lập hai node master và worker.
### Tại máy Master và Worker
##### Đăng nhập vỡi quyền tài khoản `root`
```
sudo su -
```
Thực hiện tất cả các lệnh với quyền root, trừ khi được chỉ định khác
##### Disable Firewall
```
ufw disable
```
##### Disable swap
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Cập nhật lại cài đặt của sysctl cho mạng Kubernetes 
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
##### Install docker engine
```
{
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update
  apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
}
```
### Thiết lập Kubernetes
##### Add Apt repository
```
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}
```
##### Install Kubernetes components
```
apt update && apt install -y kubeadm=1.18.5-00 kubelet=1.18.5-00 kubectl=1.18.5-00
```

### Tại máy kmaster
##### Khởi tạo Kubernetes Cluster
```
kubeadm init --apiserver-advertise-address=172.31.87.187 --pod-network-cidr=192.168.0.0/16  --ignore-preflight-errors=all
```
Với __--apiserver-advertise-address__ là địa chỉ của máy Master
##### Deploy Calico network
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```

##### Tạo lệnh tham gia Cluster
```
kubeadm token create --print-join-command
```

##### Để có thể chạy lệnh kubectl với tư cách người dùng không phải root
Nếu muốn chạy các lệnh kubectl với tư cách người dùng không phải root, thực hiện các lệnh sau 
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### On Kworker
##### Join the cluster
Sử dụng kết quả trả về từ lệnh __kubeadm token create__ ở bước lấy được ở master và chạy.

### Xác minh cluster (Trên máy master kmaster)
##### Get Nodes status
```
kubectl get nodes
```
![](https://github.com/ngoclam9200/DTDM/blob/master/File/anh%20readme/getnodes.png)
##### Get component status
```
kubectl get cs
```

## Install Kubernetes Dashboard
Kubernetes Dashboard là một giao diện người dùng dựa trên web cung cấp thông tin về trạng thái của Kubernetes Cluster resources và bất kỳ lỗi nào có thể xảy ra. Dashboard có thể được sử dụng để triển khai các ứng dụng container đến Cluster, khắc phục sự cố các ứng dụng đã triển khai, cũng như quản lý chung các  cluster resources.
### Cấu hình kubectl
Ta sử dụng công cụ quản lý kubectl kubernetes để triển khai dashboard đến Kubernetes cluster. Bạn có thể cấu hình kubectl bằng hướng dẫn dưới đây.
```
https://computingforgeeks.com/manage-multiple-kubernetes-clusters-with-kubectl-kubectx/
```
### Triển khai Kubernetes Dashboard
Việc triển trai dashboard mặc định chứa một tập hợp quyền __RBAC__ để chạy. Bạn có thể triển khai Kubernetes Dashboard bằng lệnh bên dưới.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml
```
Bạn cũng có thể tải xuống và áp dụng file local:
```
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml
kubectl apply -f recommended.yaml
```
#### Đặt Dịch vụ để sử dụng NodePort
Các dịch vụ chỉ có sẵn trên __ClusterIPs__ như có thể được nhìn thấy từ đầu ra bên dưới:
```
$ kubectl get svc -n kubernetes-dashboard

```
![](https://github.com/ngoclam9200/DTDM/blob/master/File/anh%20readme/getsvcdashboard.png)
Patch các service để nghe trên NodePort:
```
kubectl --namespace kubernetes-dashboard patch svc kubernetes-dashboard -p '{"spec": {"type": "NodePort"}}'
```
Xác nhận cài đặt mới:
```
$ kubectl get svc -n kubernetes-dashboard kubernetes-dashboard -o yaml
...
spec:
  clusterIP: 10.108.11.22
  clusterIPs:
  - 10.108.11.22
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30506
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```


Kiểm tra deployment status:
```


$ kubectl get deployments -n kubernetes-dashboard                              
```
![](https://github.com/ngoclam9200/DTDM/blob/master/File/anh%20readme/getdeploydashboard.png)

Hai Pods được tạo

```
$ kubectl get pods -n kubernetes-dashboard
```
![](https://github.com/ngoclam9200/DTDM/blob/master/File/anh%20readme/getpoddashboard.png)


Xác nhận xem dịch vụ có thực sự được tạo hay không:

```
$ kubectl get service -n kubernetes-dashboard  
```

### Truy cập Kubernetes Dashboard
Việc triển khai Dịch vụ được chỉ định một cổng 30513 / TCP.
```


```

![](https://github.com/ngoclam9200/DTDM/blob/master/File/anh%20readme/dashboardkubernetes.png)
Hãy xác nhận xem quyền truy cập vào Dashboard có hoạt động hay không.

## NGINX on a Kubernetes Cluster
Chúng tôi tạo deployment NGINX bằng cách sử dụng hình ảnh NGINX.
```
# kubectl create deployment nginx --image=nginx
```
Bây giờ bạn có thể thấy trạng thái triển khai của mình.
```
# kubectl get deployments
```
Nếu bạn muốn xem chi tiết hơn về việc triển khai của mình, bạn có thể chạy lệnh __describe__ 
Ví dụ, có thể xác định có bao nhiêu bản sao của deployment đang chạy. Trong trường hợp này,Ta mong đợi thấy một bản sao của 1 đang chạy (tức là 1/1 bản sao).
```
# kubectl describe deployment nginx
```
![Tên ảnh](https://www.tecmint.com/wp-content/uploads/2020/02/Check-Nginx-Deployment-Details.png)
Bây giờ Nginx deployment của bạn đang hoạt động, bạn có thể muốn hiển thị dịch vụ NGINX với một IP công cộng có thể truy cập được trên internet.
### Đưa dịch vụ Nginx lên publuc network
Kubernetes cung cấp một số tùy chọn khi tiếp xúc với dịch vụ dựa trên một tính năng gọi là kubernetes Service-types:
* ClusterIP – Loại Dịch vụ này thường phơi bày dịch vụ trên IP nội bộ, chỉ có thể tiếp cận trong cluster và chỉ có thể trong các cluster-nodes.
* NodePort – Đây là tùy chọn cơ bản nhất để lộ dịch vụ của bạn có thể truy cập bên ngoài cluster của bạn, trên một port cụ thể (được gọi là NodePort)trên mọi node trong cluster. Chúng tôi sẽ sớm minh họa cho lựa chọn này.
* LoadBalancer – Tùy chọn này tận dụng các dịch vụ cân bằng tải bên ngoài được cung cấp bởi các nhà cung cấp khác nhau để cho phép truy cập vào dịch vụ của bạn. Đây là một lựa chọn đáng tin cậy hơn khi nghĩ về tính khả dụng cao cho dịch vụ của bạn và có nhiều tính năng hơn ngoài quyền truy cập mặc định.
* ExternalName – Dịch vụ này thực hiện chuyển hướng lưu lượng truy cập đến các dịch vụ bên ngoài cluster. Do đó, dịch vụ được ánh xạ thành tên DNS có thể được lưu trữ ra khỏi cluster của bạn. Điều quan trọng cần lưu ý là điều này không sử dụng proxy.
Loại Dịch vụ mặc định là __ClusterIP__
Trong trường hợp này, ta muốn sử dụng loại Dịch vụ NodePort vì ở đây có cả địa chỉ public IP và private và ta không cần bộ cân bằng tải bên ngoài cho lúc này. Với loại dịch vụ này, Kubernetes sẽ chỉ định dịch vụ này trên các port trên phạm vi hơn 30000.
```
# kubectl create service nodeport nginx --tcp=80:80
```
Chạy lệnh get svc để xem tóm tắt về dịch vụ và các port được hiển thị.
```
# kubectl get svc
```
Bây giờ bạn có thể xác minh rằng trang Nginx có thể truy cập được trên tất cả các node bằng cách sử dụng lệnh __curl__.
```
# curl master-node:30386
# curl node-1:30386
# curl node-2:30386
```
![Xác nhận nginx](https://www.tecmint.com/wp-content/uploads/2020/02/Check-Nginx-Page-on-Kubernetes-Cluster.png)
Như bạn có thể thấy, “WELCOME TO NGINX!” hiển thị trên trang.
### Tiếp cận địa chỉ ip public tạm thời
Như thể nhận thấy, __Kubernetes__ báo cáo rằng tôi không có public IP nào đang hoạt động được đăng ký, hay đúng hơn là không có internal IP được đăng ký.
```
kubectl get svc
```
![get svc](https://github.com/ngoclam9200/DTDM/blob/master/File/anh%20readme/nginxgetsvc.png)
Hãy xác minh xem điều đó có thực sự đúng hay không, rằng tôi không có interal IP nào được gắn vào các giao diện của mình bằng lệnh IP.
```

![](https://github.com/ngoclam9200/DTDM/blob/master/File/anh%20readme/nginxdemo.png)



