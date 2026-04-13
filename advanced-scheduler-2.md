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


