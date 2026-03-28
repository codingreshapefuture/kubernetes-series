Env là các biến hệ thống được inject vào container runtime giúp truyền thông tin động:
+ Hardcode value: biến tĩnh
+ Configmap/secret: lấy từ object bên ngoài
+ downward api: metadata từ k8s
+ image: env mặc định từ dockerfile

Environment variables trong YAML:
```
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: my-container
    image: busybox
    env:  # Mảng các env var
    - name: STATIC_VAR  # Biến tĩnh
      value: "hello world"  # hardcode trực tiếp
    - name: CONFIG_VAR
      valueFrom:
        configMapKeyRef:  # Từ ConfigMap
          name: my-configmap
          key: app.key
          optional: false  # error nếu không tồn tại
    - name: SECRET_VAR
      valueFrom:
        secretKeyRef:  # Từ Secret
          name: my-secret
          key: password
    - name: POD_NAME  # Downward API
      valueFrom:
        fieldRef:
          fieldPath: metadata.name  # Các field: metadata.name, status.podIP, spec.nodeName, etc.
    - name: CPU_LIMIT  # Resource field
      valueFrom:
        resourceFieldRef:
          containerName: my-container  # Optional
          resource: limits.cpu
    envFrom:  # Inject toàn bộ ConfigMap/Secret làm env vars (prefix optional)
    - configMapRef:
        name: my-configmap
    - secretRef:
        name: my-secret
        prefix: DB_  # Optional, prefix keys
```

----

ConfigMap dùng để truyền cấu hình (configuration) của ứng dụng vào bên trong container, hoặc sensitive data bằng Secret.
K8s có cung cấp cho chúng ta cách truyền một list env vào bên trong từng container của Pod, nhưng khi ứng dụng ta càng lớn cần càng nhiều env hơn hoặc sử dụng lại nhiều nơi thì ConfigMap để giải quyết vấn đề đó.

Tạo ConfigMap:
```
kubectl create configmap my-cm --from-literal=key1=value1 --from-literal=key2=value2
```

Sử dụng yaml:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
data:
  DB: postgres
  USER: postgres
  PASSWORD: postgres
```

Xem thông tin configmap đã tạo:
```
kubectl describe configmap postgres-config
```

Sử dụng ConfigMap để nhập env trong deploy/pod
```
spec:
  containers:
    - image: 080196/hello-cm
      name: hello-cm
      ports:
        - containerPort: 3000
      envFrom: # using envFrom instead of env
        - configMapRef: # referencing the ConfigMap
            name: postgres-config # name of the ConfigMap
          prefix: POSTGRES_ # All environment variables will be prefixed with POSTGRES_
      env:
        - name: PORT
          value: "3000"
        - name: DB_HOST
          value: postgres
```
Có thể gọi nhiều nơi
```
spec:
  containers:
    - image: postgres
      name: postgres
      ports:
        - containerPort: 5432
      envFrom:
        - configMapRef:
            name: postgres-config
          prefix: POSTGRES_
```

Dùng ConfigMap để truyền cấu hình dạng file vào trong container thông qua volume config

Ngoài ra, ta có thể sử dụng key với value là nội dung của toàn bộ một file config, như sau:

Ví dụ:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  my-nginx-config.conf: |
    server {
      listen 80;
      server_name www.kubia-example.com;

      gzip on;
      gzip_types text/plain application/xml;

      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
    }
```

Mount config file vào container
```
spec:
  containers:
    - image: nginx:alpine
      name: web-server
      volumeMounts:
        - mountPath: /etc/nginx/conf.d # mount content of the configmap to container
          name: config
          readOnly: true
  volumes:
    - name: config # volume use configmap as content
      configMap:
        name: nginx-config # configmap name
```

---

Secret cũng giống như ConfigMap, dữ liệu được lưu dưới dạng key/value pairs, cách sử dụng tương tự.
Khác ở chỗ là nó được dùng để chứa sensitive data, được mã hóa base64 và chỉ administrator cho phép mới có thể đọc.

Có 1 số loại secret cơ bản như:
+ Opaque
+ kubernetes.io/service-account-token
+ kubernetes.io/dockercfg
+ kubernetes.io/dockerconfigjson
+ kubernetes.io/basic-auth
+ kubernetes.io/ssh-auth
+ kubernetes.io/tls
+ bootstrap.kubernetes.io/token

Tạo một secret:
```
kubectl create secret generic postgres-secret --from-literal=DB=postgres --from-literal=USER=postgres --from-literal=PASSWORD=postgres
```

Sử dụng yaml:
```
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
data:
  PASSWORD: c3VwZXJzZWNyZXQ=  # echo -n 'supersecret' | base64
  API-KEY: YXBpa2V5Cg==  
stringData:  # Plain text, Kubernetes sẽ encode
  DB: postgres
  USER: postgres
```

Khi ta xem thông tin secret sẽ thấy nó được lưu ở dạng base64 encode
```
kubectl describe secret postgres-secret
```

Sử dụng Secret trong deployment/pod
```
spec:
  containers:
  - image: 080196/hello-cm
    name: hello-cm
    ports:
    - containerPort: 3000
    envFrom: # use envFrom instead of env
    - secretRef: # use secretRef instead of env configMapRef
      name: postgres-secret # name of the Secret
      prefix: POSTGRES_
```
