# MinIO Config

ติดตั้ง MinIO
เวอร์ชั่นล่าสุด https://min.io/download#/linux
```
wget https://dl.min.io/server/minio/release/linux-amd64/minio_20220901235336.0.0_amd64.deb
dpkg -i minio_20220901235336.0.0_amd64.deb
```

สร้าง Group & User & Drive
```
sudo groupadd -r minio-user
sudo useradd -M -r -g minio-user minio-user
sudo mkdir /mnt/buckets/
sudo chown minio-user:minio-user /mnt/buckets
```

สร้างและแก้ไขไฟล์ /etc/default/minio : `sudo nano /etc/default/minio`
```
MINIO_VOLUMES="[กำหนด path สำหรับเก็บข้อมูล]"
MINIO_OPTS="--certs-dir /root/certs/ --address domain.local:443 --console-address domain.local:9001"
MINIO_ROOT_USER=[กำหนด user root]
MINIO_ROOT_PASSWORD=[กำหนด password root]
```

ให้ MinIO สามารถใช้งาน port 443 ได้ ถ้าไม่ทำจะ start service ไม่ได้
```
sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/minio
```

ติดตั้ง SSL
```
sudo mkdir /root/certs/
sudo chown minio-user:minio-user /root/certs/public.crt
sudo chown minio-user:minio-user /root/certs/private.key
```

จัดการ Service
```
sudo systemctl start minio    | sudo service minio start
sudo systemctl status minio   | sudo service minio status
sudo systemctl restart minio  | sudo service minio restart
sudo systemctl stop minio     | sudo service minio stop
```

กรณีใช้ nginx

```
server {
	listen 443 ssl;
	server_name domain.local;

	ignore_invalid_headers off;
	client_max_body_size 0;
	proxy_buffering off;

	ssl_certificate /etc/nginx/ssl/public-star.crt;
	ssl_certificate_key /etc/nginx/ssl/private.key;
	ssl_client_certificate /etc/nginx/ssl/ca.crt;
	#ssl_verify_client optional;

	location ~ /$ {
		return 301  https://$host:19001$request_uri;
	}

	location / {
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header Host $http_host;

		proxy_connect_timeout 300;
		proxy_http_version 1.1;
		proxy_set_header Connection "";
		chunked_transfer_encoding off;

		proxy_pass http://localhost:9000;
	}
}
	
server {
	listen 19001 ssl;
	server_name domain.local;

	ignore_invalid_headers off;
	client_max_body_size 0;
	proxy_buffering off;

	ssl_certificate /etc/nginx/ssl/public-star.crt;
	ssl_certificate_key /etc/nginx/ssl/private.key;
	ssl_client_certificate /etc/nginx/ssl/ca.crt;
	#ssl_verify_client optional;

	location / {
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header Host $http_host;

		proxy_connect_timeout 300;
		proxy_http_version 1.1;
		proxy_set_header Connection "";
		chunked_transfer_encoding off;

		proxy_pass http://localhost:9002;
	}
}
```

- https://www.digitalocean.com/community/tutorials/how-to-set-up-minio-object-storage-server-in-standalone-mode-on-ubuntu-20-04
- https://www.digitalocean.com/community/tutorials/how-to-set-up-an-object-storage-server-using-minio-on-ubuntu-16-04
