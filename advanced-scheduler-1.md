## K8s advanced scheduling

+ node-affinity
+ pod-autoscale
+ request-limit
+ tain-tolerations
+ rolling-update
+ rollout-rollback

---

### node-affinity

Có 2 cách phổ biến nhất để schedule pod hoặc deployment dựa trên label của worker Node mong muốn là node selector và node affinity

Xem labels của các nodes:
kubectl get nodes --show-labels

Đánh label cho nodes:
kubectl label node <node-name> <key>=<value>
kubectl label node <node-name> disktype=ssd
kubectl label node <node-name> cpu=high

=== Node Selector

Node selector dùng để chỉ định trực tiếp label của node để sử dụng deployment
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-selector
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: main
      image: busybox
      command: ["dd", "if=/dev/zero", "of=/dev/null"]
```

Tuy nhiên, nó phải chọn chính xác theo label hoặc nếu node đã đủ số lượng pod thì sẽ không thể deploy nữa

=== Node Affinity

Node affinity cho phép chọn theo Expression linh hoạt hơn, nhưng cú pháp sẽ dài dòng hơn chút
+ nodeSelectorTerms: mảng chứa nhiều matchExpressions
+ operator: In, NotIn, Exists, DoesNotExist, Gt, Lt
+ requiredDuringSchedulingIgnoredDuringExecution: pod chỉ được schedule khi node đã có label đó, không ảnh hưởng các pod đã ở node trước đó
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-ssd
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: disktype
              operator: In
              values:
                - "ssd"
  containers:
    - name: main
      image: busybox
      command: ["dd", "if=/dev/zero", "of=/dev/null"]
```

Ngoài ra còn có lựa chọn theo độ ưu tiên để đặt các rule tùy chỉnh hơn:
+ preferredDuringSchedulingIgnoredDuringExecution: pod sẽ ưu tiên schedule với các node này hơn, được đặt theo trọng số
+ weight: trọng số ưu tiên, càng cao càng ưu tiên trước
```
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 50
          preference:
            matchExpressions:
            - key: cpu
              operator: In
              values:
                - "high"
        - weight: 30
          preference:
            matchExpressions:
            - key: cpu
              operator: In
              values:
                - "medium"
        - weight: 20
          preference:
            matchExpressions:
            - key: cpu
              operator: In
              values:
                - "normal"
```

Xem các pod đang chạy trên node nào
kubectl get pods -owide

---

### pod-autoscale

K8s AutoScaling có 2 loại
+ HorizontalPodAutoscaler : tăng thêm tài nguyên trực tiếp
+ VerticalPodAutoscaler: tăng thêm số lượng node

Ngoài ra, nếu muốn scale theo custom metrics thì có thể tìm hiểu KEDA, Knative

Tham khảo: https://kubernetes.io/docs/concepts/workloads/autoscaling/

Để dùng K8s Auto Scale thì cần:
+ Metrics Server
+ Resource requests
+ Deployment/ReplicaSet

Cài đặt Metrics Server (sử dụng Kubernetes Metrics API)
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Kiểm tra tài nguyên đang sử dụng từ API Server:
```
kubectl top
kubectl top node
kubectl top pod
```

Tạo HPA nhanh bằng lệnh:
```
kubectl autoscale deployment ecomerce-backend-deployment --cpu-percent=70 --min=2 --max=5
kubectl get hpa
```

Tạo HPA bằng yaml:
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: ecomerce-backend-autoscaling
spec:
  scaleTargetRef: # tham chiếu đối tượng deployment
    apiVersion: apps/v1
    kind: deployment
    name: ecomerce-backend-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # CPU > 70% => scale
```

-> Khi pod vượt quá yêu cầu đề ra trong 15s sẽ tự động scale thêm
-> Khi pod giảm dưới yêu cầu đề ra trong 300s sẽ tự động scale xuống

HPA với RAM (ít dùng), QPS (Query per seconds)
```
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue # tổng memory / số Pod
        averageValue: 500Mi
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second # từ Prometheus
      target:
        type: AverageValue
        averageValue: 100
```
