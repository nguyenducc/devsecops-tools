# Cài đặt Kubespray
## 1. Cài đặt server
Tạo các VM với cấu hình sau:
| Hostname | Ip | RAM | CPU | Disk | OS |
| :----- | :---------- | :-------------- | :-------------- | :--------------| :--------------| 
| master-01      | 192.168.56.111           | 3GB                | 2 | 20    GB| ubuntu-server 24.04                 
| master-02     | 192.168.56.112           | 2GB               | 2|20GB|ubuntu-server 24.04
| master-03      | 192.168.56.113           | 2GB               | 2|20GB|ubuntu-server 24.04
## 2. Enable ssh cho user root và add host trên các server (thực hiện lần lượt trên từng server) 
Truy cập user root và chỉnh sửa cấu hình ssh:
```sh
sudo -i
vi /etc/ssh/sshd_config
```
Tìm dòng **#PermitRootLogin prohibit-password** thay đổi thành:
```sh
PermitRootLogin yes
```
### Cài đặt mật khẩu cho user root:
Trên màn hình terminal thực hiện câu lệnh sau và chọn mật khẩu cho user root:
```sh
passwd
```
### Add host cho các server:
```sh
vi /etc/hosts
```
<div style="page-break-after: always;"></div>

Thêm các cấu hình sau:
```sh
192.168.56.111 master-01
192.168.56.112 master-02
192.168.56.113 master-03
```
### Khởi động lại và truy cập ssh với user root
```sh
reboot
ssh root@<<ip tương ứng với server vừa reboot>>
```
###  Tắt swap để không bị reset cấu hình khi reboot
```sh
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

## 3. Cài đặt Kubespray v1.28 (version mới nhất hiện tại) tương ứng release 2.24
### 1. Cài ansible 2.14+ cho node master.

Theo tài liệu chính thức của kubespray thì cần cài version ansible version 2.14 trở lên để tương thích với kubespray v1.28.
Ở đây lựa chọn node master-01 làm master node mặc định và bước này chỉ thao tác trên node master.

Update các package trước khi cài đặt.
```sh
apt update -y && apt upgrade -y
```
Theo tài liệu chính thức để cài đặt và sử dụng Ansible thì cần có python3.
```sh
apt install git python3 python3-pip -y
```

### 1. Clone source code của Kubespray:
```sh
git clone https://github.com/kubernetes-sigs/kubespray.git -b release-2.24
```

Cài đặt Ansible:
**Cách 1: sử dụng ppa**
```sh
sudo apt-add-repository ppa:ansible/ansible
apt update -y
apt install ansible-core -y
```
hoặc 
**Cách2: sử dụng venv
```sh
VENVDIR=kubespray-venv
KUBESPRAYDIR=kubespray
python3 -m venv $VENVDIR
source $VENVDIR/bin/activate
cd $KUBESPRAYDIR
pip install -U -r requirements.txt
```
Thoát khỏi pyenv
```sh
alias ansible="~/kubespray-venv/bin/ansible"
echo "alias ansible='~/kubespray-venv/bin/ansible'" >> ~/.bashrc
alias ansible-playbook="~/kubespray-venv/bin/ansible-playbook"
echo "alias ansible-playbook='~/kubespray-venv/bin/ansible-playbook'" >> ~/.bashrc
```
Tải lại file ~/.bashrc:
```sh
source ~/.bashrc
```

*Ưu tiên sử dụng virtual env* để cài đúng version 

Kiểm tra đã cài hoàn thành chưa:
```sh
ansible --version
```
### 2. Generate ssh key cho các worker node.
Chạy câu lệnh gen key và enter đến khi kết thúc:
```sh
ssh-keygen -t rsa
```
Copy key sang các server tương ứng:

```sh
ssh-copy-id 192.168.56.111
ssh-copy-id 192.168.56.112
ssh-copy-id 192.168.56.113
```
Kiểm tra xem đã ssh vào user root không cần password thành công chưa:
```sh
ssh root@192.168.56.112
```

### 3. Setup cụm K8S
```sh
cd kubespray/
cp -rfp inventory/sample inventory/mycluster
vi inventory/mycluster/hosts.ini
```

<div style="page-break-after: always;"></div>

Copy nội dung sau vào file host.ini
```sh
## Configure 'ip' variable to bind kubernetes services on a
## different ip than the default iface
[all]
master-01 ansible_host=192.168.56.111 ip=192.168.56.111
master-02 ansible_host=192.168.56.112 ip=192.168.56.112
master-03 ansible_host=192.168.56.113 ip=192.168.56.113

[kube-master]
master-01
master-02
master-03

[etcd]
master-01

[kube-node]
master-01
master-02
master-03

[k8s-cluster:children]
kube-node
kube-master

[calico-rr]

[vault]
master-01

```
Chạy câu lệnh sau để hoàn thành setup cụm:
```sh
ansible-playbook -i inventory/mycluster/hosts.ini  --become --become-user=root cluster.yml
```
Kiểm tra cụm đã setup thành công chưa:
```sh
kubectl get no
```
