K8s storage có thể được cung cấp qua nhiều cách:
- storageclass/CSI dynamic provisoning volume
- pv/pvc cung cấp volume cho pod
- pod sử dụng volume mount trực tiếp

Mối quan hệ:
```
Storage backend (disk, NFS, cloud, etc.) → StorageClass (optional) → PV (cấp phát) → PVC (yêu cầu) → Pods Volumes (sử dụng)
```

--- Volumes

Tham khảo: https://kubernetes.io/docs/concepts/storage/volumes/

Volume có thể dùng với mục đích:
+ share volume chia sẻ dữ liệu giữa các container trong Pod
+ mount filesystem của một node
+ lưu trữ dữ liệu không bị mất khi pod restart (PV)

Một số loại volume phổ biến trong K8s:
+ emptyDir
+ hostPath
+ gitRepo
+ gcePersistentDisk, awsElasticBlockStore, azureDisk (cloud storage)
+ configMap, secret, downwardAPI
+ PersistentVolumeClaim, PersistentVolumes

--- PersistentVolume (PV) / PersistentVolumeClaim (PVC)

PV/PVC là một tài nguyên lưu trữ thực tế trong cluster, được tạo ra bởi admin hoặc provision tự động

PV/PVC tách biệt rõ giữa người cung cấp (admin) và người dùng (developer)
-> K8s administrator sẽ setup kiến trúc storage bên dưới và tạo PV
-> còn deverloper chỉ cần quan tâm size của volume request từ PVC storage

Tham khảo: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

Các loại PV:
+ csi/fc/hostpath/iscsi/local/nfs
+ cloud storage (aws, azure, cinder, flex, gce, portworx, vsphere)
+ other (cephfs/flocker/glusterfs/photon/quobyte/rbd)

PVC sẽ tự động chọn PV phù hợp theo thứ tự ưu tiên:
+ storageClassName định nghĩa trong PV hoặc StorageClass có sẵn (nếu không chỉ định mặc định tìm tất cả PV không định nghĩa)
+ PV size khả dụng nhiều hơn

Access mode: (lưu ý mỗi loại PV sẽ có khả năng khác nhau)
+ RWO: ReadWriteOnce
+ ROX: ReadOnlyMany
+ RWX: ReadWriteMany
+ RWOP: ReadWriteOncePod

Reclaim Policy khi pod bị xóa:
+ Retain: Giữ PV lại sau khi PVC xóa
+ Delete: Xóa PV cùng với PVC
+ Recycle: Dọn sạch data rồi reuse

--- StorageClass

StorageClass là một đối tượng định nghĩa lớp trừu tượng cho các loại lưu trữ bên ngoài (storage backend)

Storage class giúp dynamic provisioning (tự động tạo PV khi có PVC) -> thường có trên cloud, trên local phải cài provisioner

Tham khảo: https://kubernetes.io/docs/concepts/storage/storage-classes/

Các loại Provisioner:
+ local/nfs/RBD
+ cloud storage (AzureFile, Portworx, Vsphere)
