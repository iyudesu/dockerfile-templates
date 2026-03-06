# dockerfile-templates
To store usable Dockerfile template in most used programming languages

To test Dockerfile

1. Go to desired code
```sh
cd <directory>
```

2. Build 
```sh
docker buildx build --platform linux/amd64,linux/arm64 \     
  --build-arg VERSION=<major.minor.patch> \
  --build-arg COMMIT=$(git rev-parse --short HEAD) \
  -t <image>:<tag> .
```

3. Run
```sh
docker run -it -d -p <node-port>:<container-port> --name <container> <image>:<tag>
```