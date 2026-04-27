Ingress giúp expose traffic HTTP/HTTPS từ bên ngoài đến các dịch vụ trong cụm. Traffic routing được kiểm soát bởi các quy tắc trên tài nguyên Ingress

Lưu ý: Ingress không phải Service, mà là tập hợp rule được Ingress Controller thực thi

Ingress controller chịu trách nhiệm thực hiện Ingress, như load balance, name-based virtual host,...
+ Với baremetal thì Ingress controller sử dụng NodePort + loadbalancer server để expose
+ Với cloud thì Ingress controller có thể sử dụng Loadbalancer trực tiếp luôn

Ngoài ra, Ingress sử dụng annotation để chỉ định những cấu hình đặc biệt như whitelist, blacklist,...
-> Annotations phụ thuộc vào controller (VD: nginx.ingress.kubernetes.io/...)

Tham khảo: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

---

### Demo HTTP/80/nginx

Tham khảo: https://kubernetes.github.io/ingress-nginx/deploy/

Có thể cài Nginx Ingress Controller bằng nhiều cách:
+ Helm chart
+ YAML manifests
+ Local cluster addons (minikube, microk8s, docker, rancher)
+ Cloud

Cài đặt với Helm:
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx --create-namespace --namespace ingress-nginx
```

Cài đặt với YAML manifests:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml
# kubectl apply -f https://raw.githubusercontent.com/kubernetes/nginx-ingress/main/deploy/static/provider/cloud/deploy.yaml
```

Service nginx-controller-admission sử dụng NodePort, hoặc LoadBalancer để lấy IP Public (mặc định)

`http://<ip-node>:<node-port-web>` -> `http://<ip-loadbalancer>:80`

Kiểm tra:
```
kubectl get all -n ingress-nginx
kubectl get ingressclass
```

Tạo demo app và service:
```
kubectl create deployment demo-app --image=httpd --port=80
kubectl expose deployment demo-app
```

Tạo ingress rule:
```
kubectl create ingress demo-ingress --class=nginx --rule="dev.local/*=demo-app:80"
kubectl get ingress
```

Test:
```
curl --resolve dev.local:80:<ip-loadbalancer> http://dev.local
```

Thêm DNS pointer vào file hosts
```
curl http://dev.local
```

Kết quả sẽ là
```
<html><body><h1>It works!</h1></body></html>
```

---

### Demo HTTPS/8080/haproxy

Tham khảo: https://www.haproxy.com/documentation/kubernetes-ingress/community/installation/on-prem/

Có thể cài Haproxy Ingress Controller bằng nhiều cách:
+ Helm chart
+ YAML manifests
+ Cloud, Rancher

Cài đặt với Helm:
```
helm repo add ingress-haproxy https://haproxytech.github.io/helm-charts
helm repo update
helm install ingress-haproxy ingress-haproxy/kubernetes-ingress --create-namespace --namespace ingress-haproxy
```

Cài đặt với YAML manifests:
```
kubectl apply -f https://raw.githubusercontent.com/haproxytech/kubernetes-ingress/master/deploy/ingress-haproxy.yaml
```

Mặc định service ingress-haproxy sử dụng NodePort, đổi về LoadBalancer để lấy IP Public

`http://<ip-node>:<node-port-web>` -> `http://<ip-loadbalancer>:80`

Kiểm tra:
```
kubectl get all -n ingress-haproxy
kubectl get ingressclass
```

Tạo demo app và service:
```
kubectl create deployment echoserver --image k8s.gcr.io/echoserver:1.3
kubectl expose deployment echoserver --port=8080
```

Tạo ingress rule:
```
kubectl create ingress demo-ingress --class=haproxy --rule="echo.local/*=echoserver:8080,tls"
kubectl get ingress
```

Test:
```
curl -k --resolve echo.local:443:<ip-loadbalancer> https://echo.local
```

Nếu curl qua IP không qua domain sẽ trả về page Nginx trắng/Not found

Thêm DNS pointer vào file hosts
```
curl -k https://echo.local
```

Kết quả thông tin client kết nối sẽ được trả về

