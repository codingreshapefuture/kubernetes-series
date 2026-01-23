```sh
kubectl config view: xem cấu hình kubernetes
kubectl cluster-info: xem trạng thái của cluster hiện tại

kubectl get nodes -owide: xem trạng thái các nodes trong cụm
kubectl get nodes --show-labels: xem trạng thái các nodes và labels
kubectl get pod --all-namespaces: xem thông tin tất cả các pod hệ thống của k8s
kubectl get pod -A: xem thông tin tất cả các pod hệ thống của k8s

kubectl get pod -n <namespace>: xem thông tin các pod theo namespace
kubectl get pod -l <label>: xem thông tin các pod theo label

kubectl get ns: danh sách các namespaces
kubectl get pod: danh sách tất cả các pods
kubectl get svc: danh sách tất cả các services
kubectl get deploy: danh sách tất cả các deployment
kubectl get rs: danh sách tất cả các replicaset
kubectl get sc: danh sách các storageclass
kubectl get pv: danh sách các persistentvolumes
kubectl get pvc: danh sách các persistentvolumeclaims
kubectl get cm: danh sách các configmaps
kubectl get secret: danh sách các secrets

kubectl describe deployment <NAME>: xem thêm nhiều thông tin khác của deloyment/pod như các event và status
kubectl get deployment -o wide: xem thông tin về deployment như replicas, image
kubectl get deployment -o yaml: xem thông tin về deployment config dưới dạng yaml

kubectl create deployment <NAME> --image=<image>:<tag>: dùng để tạo deployment (blueprint)
kubectl edit deployment <NAME>: dùng để sửa thông tin deployment trực tiếp dưới dạng yaml
kubectl delete deployment <NAME>: dùng để xóa deployment (pod và replicaset cũng bị xóa theo)

kubectl scale deployments <NAME> --replicas=<number>: dùng để scale deployment và replicaset lên nhanh chóng
kubectl autoscale deployment <NAME> --cpu-percent=80 --min=2 --max=5: dùng để HorizontalPodAutoscaler cho deployment dựa trên thông số cpu

kubectl apply -f <FILE_NAME>: tạo resource trong K8s từ file yaml config
kubectl delete -f <FILE_NAME>: xóa resource trong K8s từ file yaml config

kubectl exec -it <POD_NAME> -- /bin/bash: ta sẽ inject cotainer của pod để tiến hành chạy command trong đó.

kubectl logs <POD_NAME>: xem logs của pod

kubectl port-forward <POD_NAME> <port out:port in>: forward traffic nội bộ ra bên ngoài
kubectl expose deployments <NAME> --type=<service-type> --port=80 --target-port=8080: tạo service LoadBalancer điều hướng port 80 traffic đến deployment port 8080

kubectl top pod: xem thông số tiêu thụ cpu/ram của pod
kubectl top node:  xem thông số tiêu thụ cpu/ram của node
kubectl get resourcequota: xem tổng dung lượng tài nguyên của namespace

kubectl set image deployment/<NAME> nginx=<image>:<tag>: cập nhật image của deployment (trigger rollout)
kubectl rollout status deployment/my-dep: xem trạng thái rollout của deployment
kubectl rollout history deployment/<NAME>: xem lịch sử rollout của deployment
kubectl rollout undo deployment/<NAME> --to-revision=<number>: rollback deployment về phiên bản cụ thể
```
