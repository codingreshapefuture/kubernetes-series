## So sánh Pod vs Deploy vs RS

Pod là đơn vị triển khai nhỏ nhất của K8s, nó như một môi trường ảo độc lập có thể chứa 1 hoặc nhiều container

-> Thông thường sẽ không triển khai pod đơn lẻ mà thường sẽ sử dụng workload management (deploy,statefulset,daemonset,...)

Deployment là file để định nghĩa chiến lược triển khai pod thường được dùng trong thực tế

-> Nó sử dụng ReplicaSet để duy trì số lượng Pods mong muốn, hỗ trợ rollout (cập nhật dần dần), rollback (quay lại phiên bản cũ), và scaling (tăng/giảm replicas)

ReplicaSet là controller duy trì số lượng Pods chính xác

-> RS là building block cho Deployment, nhưng hiếm khi dùng trực tiếp vì không hỗ trợ tính năng update/rollback

## Service

Khi bạn tạo một Service kiểu ClusterIP hoặc NodePort, kube-proxy sẽ tự động phân phối traffic đến các Pod backend theo cơ chế round-robin (ngẫu nhiên hoặc IPVS)

Tạo service ClusterIP:
```
kubectl expose deployment nginx-app --name=nginx-service --port=8080 --target-port=80
```

Kiểm tra IP Pod:
```
kubectl get endpoints nginx-service
```

Test kết nối:
```
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- http://nginx-service:8080
```

-> Có thể gọi service qua IP Internal, ClusterIP, NodePort
