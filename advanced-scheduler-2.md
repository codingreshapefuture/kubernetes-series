## K8s advanced scheduling

+ node-affinity
+ pod-autoscale
+ request-limit
+ tain-tolerations
+ rolling-update
+ rollout-rollback

---

### request-limit

Request và Limit giúp xác định giới hạn tài nguyên cung cấp cho Pod

+ Request: đảm bảo container có tài nguyên đặt chỗ tối thiểu để chạy
+ Limit: giới hạn tài nguyên tối đa container có thể sử dụng

Nếu quá tài nguyên cung cấp:
+ Request: quyết định khả năng schedule tới node còn dư tài nguyên
+ Limit: throttled nếu tràn CPU, kill container (OOM) nếu tràn RAM

Đơn vị:
+ CPU: 1 = 1 vCPU/core; 100m = 0.1 CPU
+ Memory: Bytes (e.g. 256Mi = 256 mebibytes, 1Gi = 1 gibibyte)

```
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

---

### tain-tolerations

Taint và tolerations trong K8s giúp ta trong việc restrict pod được deploy tới node không mong muốn
+ Taints được dùng để gán nhãn cấm pod được deploy tới worker node, tức là node đó "không muốn" nhận pod
+ Tolerations được dùng để khai báo pod có thể "chịu đựng", giúp deploy lên những worker node mà có gán taints

Theo mặc định, master node được đánh taints không cho deploy pod lên master node mà trừ những system pod có tolerations phù hợp
```
kubectl describe node kube-master
```

Taints dùng như đánh label:
```
kubectl taint node <node-name> <key>=<value>:<effect>
kubectl taint node production-node node-type=production:NoSchedule
```

Có 3 giá trị Taint effect:
+ NoSchedule: không cho phép pod được schedule lên node, pod đang chạy trên node trước đó không bị ảnh hưởng.
+ NoExecute: ảnh hưởng lên cả pod mà đang chạy trên node, nếu pod không có tolerations phù hợp với taints thì nó cũng sẽ bị gỡ khỏi node.
+ PreferNoSchedule: nếu Pod không thể deploy tới bất kì node nào khác thì có thể được deploy lên nó.

Chỉ định tolerations lên pod deploy:
```
spec:
  tolerations:
    - key: node-type
      Operator: Equal
      value: production
      effect: NodeSchedule
```

Ngoài ra, có một số trạng thái đặc biệt như node bị die thì K8s sẽ đánh cho nó là
"Tolerations: node.kubernetes.io/not-ready:NoExecute op=Exists for 300s"
-> Tức sẽ chờ 300s thì sẽ xóa pod khỏi node hiện tại
