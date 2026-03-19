Khi bạn tạo một Service, Kubernetes sẽ tự động đăng ký DNS record cho Service đó trong CoreDNS

Mọi Pod trong cluster có thể truy cập Service này bằng tên DNS thay vì IP qua `<service-name>.<namespace>.svc.cluster.local`

```
kubectl describe pod helloworld
kubectl exec -it helloworld -- sh
curl helloworld:3000
```

CoreDNS là thành phần chịu trách nhiệm phân giải DNS trong cluster

Nếu DNS không resolve được, kiểm tra CoreDNS logs:
```
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

Có một số loại dnsPolicy:
+ ClusterFirst (default): Dùng DNS của cluster để resolve các Service nội bộ
+ Default: Dùng DNS cấu hình sẵn của node (thường là /etc/resolv.conf)
+ ClusterFirstWithHostNet: Dùng cho Pod có hostNetwork: true
+ None: Tắt hoàn toàn, dùng dnsConfig tùy chỉnh

```
apiVersion: v1
kind: Pod
metadata:
  name: dns-test
spec:
  containers:
  - name: test
    image: busybox
    command: ["sleep", "3600"]
  dnsPolicy: Default
```

Kiểm tra DNS Resolution:
```
kubectl run test-dns --image=busybox --rm -it --restart=Never -- nslookup kubernetes.default
```

Kết quả:
```
Server:    10.96.0.10
Address:   10.96.0.10#53
Name:      kubernetes.default.svc.cluster.local
Address:   10.96.0.1
```

---

Mặc định, khi 1 pod tạo ra sẽ có IP động và có thể giao tiếp tới các pod khác qua IP đó
-> nhưng thực tế không bao giờ kết nối trực tiếp qua Pod IP mà dùng Service Name (DNS)

Gọi service qua pod DNS
+ cùng namespace: `curl <service-name>:<port>`
+ khác namespace: `curl <service-name>.<namespace>.svc.cluster.local:<port>`

Tạo service cho pod
+ gán service cho pod sử dụng selector (nếu nhiều sẽ được loadbalancer)
+ protocol có thể sử dụng TCP/UDP/ICMP
+ có nhiều loại service sử dụng type

ClusterIP sẽ expose IP nội bộ trong pod ra IP cluster
+ các pod khác có thể giao tiếp qua `<service-name>:<cluster-port>`
+ service được truy cập trong cluster
+ có thể tạo nhiều port/target port cho service

NodePort sẽ expose IP nội bộ trong pod ra IP node
+ các pod khác có thể giao tiếp qua `<service-name>:<cluster-port>`
+ các pod khác có thể giao tiếp qua `<IP node>:<node port>`
+ service có thể truy cập ngoài internet
+ có thể tạo nhiều port/target port cho service
+ port expose ra ngoài theo range 30000-32767

LoadBalancer sẽ expose IP nội bộ trong pod ra IP LB
+ các pod khác có thể giao tiếp qua `<service-name>:<cluster-port>`
+ các pod khác có thể giao tiếp qua `<IP node>:<node port>`
+ các pod khác có thể giao tiếp qua `<IP LB>:<cluster-port>`
+ service có thể truy cập ngoài internet
+ có thể tạo nhiều port/target port cho service
+ cần sử dụng cloud LB hoặc metalLB mới dùng được

ExternalName sẽ expose service qua domain CNMAE
+ map đến DNS bên ngoài, không mở port
+ ít dùng, chưa thử

Ingress sẽ routing service qua url path
+ không expose port, map đến DNS nội bộ
+ expose service qua routing rules
Lưu ý: Ingress không phải Service, mà là tập hợp rule được Ingress Controller thực thi

