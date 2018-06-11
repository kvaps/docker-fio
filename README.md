# Containerized fio

For testing clustered storage systems

* [Massfio](https://github.com/kvaps/massfio/)
* [Fio](https://github.com/kvaps/docker-fio/)

### Deploy

**Docker run:**

```bash
docker run --rm -ti -p 8765:8765 kvaps/fio --server
```

**Kubernetes run:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: fio
spec:
  selector:
    matchLabels:
      app: fio
  serviceName: "fio"
  replicas: 1
  template:
    metadata:
      labels:
        app: fio
    spec:
      containers:
      - name: fio
        image: kvaps/fio:latest
        args: [ "--server" ]
        ports:
        - name: fio
          containerPort: 8765
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 10Gi
```

### Usage

**write config**

```
cat > test.ini <<\EOT
[readtest]
blocksize=4k
filename=/data/test_file
rw=randread
direct=1
buffered=0
ioengine=libaio
iodepth=256
runtime=120
filesize=8G
[writetest]
blocksize=4k
filename=/data/test_file
rw=randwrite
direct=1
buffered=0
ioengine=libaio
iodepth=16
runtime=120
filesize=8G
EOT
```

**Run single test**

```bash
docker run --net=host -ti --rm -v "$PWD:/data" kvaps/fio --client=10.112.0.104 /data/test.ini
```

**Run multiple test**

```bash
# Run massfio container
docker run --net=host -ti --rm -v "$PWD:/data" -v "$PWD/results:/results" kvaps/massfio

# Run test
timeout 10m /massfio.sh /data/test.ini 10.112.0.104 10.112.0.105 10.112.0.106

# Parse results
/massfio_parse.sh /results
```
