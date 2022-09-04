# MinIO Config

สร้างและแก้ไขไฟล์ /etc/default/minio
```
MINIO_VOLUMES="/mnt/data"
MINIO_OPTS="--certs-dir /root/ssl/ --address domain.local:443 --console-address domain.local:9001"
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=12345678
```

allow 443 port
```
sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/minio
```

- https://www.digitalocean.com/community/tutorials/how-to-set-up-minio-object-storage-server-in-standalone-mode-on-ubuntu-20-04
- https://www.digitalocean.com/community/tutorials/how-to-set-up-an-object-storage-server-using-minio-on-ubuntu-16-04
