# MinIO Best Practices

## Overview

- High-performance, S3-compatible, self-hosted object storage solution.
- Seamless S3 tools integration: `mc`, `aws-cli`, ...
- Easy to set up standalone server for small scale application using Docker. Distributed features supported.
- Performance optimized for large-scale data pipelines.
- License: GNU AGPL v3.0
- Use case: AI/ML, data analytics, ...

## Features

Community Edition (GNU AGPL v3):

- Basic features: Browse contents, create buckets, upload objects
- MinIO Server hosts a dedicated web UI (a.k.a MinIO Console) at `127.0.0.1:9000` with default credential: `minioadmin:minioadmin`.
- MinIO Client: `mc` CLI tool, SDK Go, Python, ...
  > IMO, just use `aws` CLI tool to avoid learning vendor-specific tool.
- Reliable parallel upgrades via `mc admin update ...` command (don't do this for container deployment, simply update the image is enough)
- No optimization, no regulatory compliants.

## Cheatsheet

```sh
# container
docker run \
  --name="minio" \
  --init \
  --rm \
  -p="9000:9000" \
  -p="9001:9001" \
  -u="1000:1000" \
  -v="./data:/data" \
  --security-opt="no-new-privileges:true" \
  --pull=always \
  quay.io/minio/minio \
  server /data --console-address ":9001"

# firewall configuration
ufw allow 9000:9001/tcp
firewall-cmd --get-active-zones
firewall-cmd --zone="<active-zones>" --add-port=9000/tcp --permanent
firewall-cmd --reload # DO NOT FORGET THIS

# iptables
iptables -A INPUT -p tcp --dport 9000:9001 -j ACCEPT
service iptables restart
```

```sh
aws configure --profile minio
# AWS Access Key ID [None]: <insert your MINIO_ROOT_USER here>
# AWS Secret Access Key [None]: <insert your MINIO_ROOT_PASSWORD here>
# Default region name [None]: ap-southeast-1
# Default output format [None]: ENTER

# enable AWS Signature Version ‘4’
aws configure --profile minio set s3.signature_version s3v4

# list buckets
aws --endpoint-url https://play.min.io:9000 s3 ls

# list content inside bucket
aws --endpoint-url https://play.min.io:9000 s3 ls s3://mybucket
```

### Setup MinIO behind a load balencer in production environment

- Domain `https://minio.example.net/` + Nginx rules forwarding traffic on server port `:9000` to the MinIO.
- Domain `https://console.minio.example.net/` + Nginx rules forwarding traffic on Console port `:9001` to the MinIO Console.

=> Run `export MINIO_BROWSER_REDIRECT_URL=https://console.minio.example.net` (the domain for Console port traffic)

### Python SDK

```py

```

## References

- [minio/minio](https://github.com/minio/minio)
