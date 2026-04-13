## K8s advanced scheduling

+ node-affinity
+ pod-autoscale
+ request-limit
+ tain-tolerations
+ rolling-update
+ rollout-rollback

---

### rolling-update

Có 2 chiến lược triển khai deployment trong K8s:
+ Rolling Update là chiến lược mặc định trong K8s, đảm bảo mỗi khi cập nhật thì version cũ vẫn giữ phiên bản hoạt động và sẽ cập nhật dần các pod
+ Recreate là chiến lược sẽ tạo lại toàn bộ pod trong deploy khi cập nhật (thường ít dùng hơn trong thực tế)

Quy trình chiến lược triển khai Rolling update
VD: replicas: 4, maxSurge: 1, maxUnavailable: 1
-> Tạo 2 Pod mới trong trạng thái ContainerCreating và 1 xóa Pod cũ đi vậy chính xác hiện tại sẽ có 2 Pod mới trong trạng thái ContainerCreating và 1 Pod Terminated. Hiện có 5 Pod.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: car-serv-deployment
  name: car-serv-deployment
  namespace: car-serv
spec:
  replicas: 2 # number of pod ensure to online
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1 # number of pods can expand beyond replicas
      maxUnavailable: 1 # number of pod can be delete
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: car-serv-deployment
  template:
    metadata:
      labels:
        app: car-serv-deployment
      namespace: car-serv
    spec:
      containers:
        - image: elroydevops/car-serv
          name: car-serv
          ports:
            - containerPort: 80
              name: tcp
              protocol: TCP
```

---

### rollout-rollback

Mỗi 1 lần apply lại deployment hoặc update trực tiếp deployment từ terminal (đổi image, volume,...) thì deployment cũng chạy lại
-> Mỗi lần chạy lại như thế được gọi là 1 lần rollout

```
kubectl rollout status -w deployment/myapp-deploy
kubectl rollout history deployment/myapp-deploy
```

Giả sử muốn rollback lại bản cũ
```
kubectl rollout undo deployment/myapp-deploy
kubectl rollout undo deployment/myapp-deploy --to-revision=2
```

Restart lại rollout nếu update nhẹ như biến môi trường, subpath mount
```
kubectl rollout restart deployment/myapp-deploy
```
