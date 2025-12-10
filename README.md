create a cluster with a registry
```
k3d cluster create mycluster --registry-create mycluster-registry:12345
```
pull an image, tag, then push
```
docker pull nginx:latest
docker tag nginx:latest k3d-registry.localhost:12345/nginx:latest
docker push k3d-registry.localhost:12345/nginx:latest
```
deploy the image
```
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test-registry
  labels:
    app: nginx-test-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-test-registry
  template:
    metadata:
      labels:
        app: nginx-test-registry
    spec:
      containers:
      - name: nginx-test-registry
        image: mycluster-registry:5000/nginx:latest
        ports:
        - containerPort: 80
EOF
```
