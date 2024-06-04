
```bash
minikube profile list
minikube stop -p <Node Name>
minikube delete -p <NOde Name>
```

```bash
nvidia-smi
```

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

```bash
docker run --rm --gpus all ubuntu:20.04 nvidia-smi
```

<!-- ```bash
sudo mkdir /etc/docker
sudo nano /etc/docker/daemon.json
```

```json
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
``` -->

```bash
minikube start --driver docker --container-runtime docker --gpus all
```
