# Kubeflow 설치 (v1.8)

[https://www.kubeflow.org/docs/releases/kubeflow-1.8/#component-versions](https://www.kubeflow.org/docs/releases/kubeflow-1.8/#component-versions)

## Docker 설치

```bash
docker version
```

```bash
sudo usermod -aG docker $USER   # Permission  denied 시, 실행 후 재시작
```

## Minikube 설치

``` bash
cd ~
mkdir Downloads
cd Downloads/
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

## Kubernetes 실행 (v1.26)
<!--
``` bash
minikube start -p kubeflow --cpus 4 --memory 8096 --kubernetes-version=v1.26 --driver=docker
``` -->

``` bash
minikube start -p kubeflow --cpus 4 --memory 8096 --driver docker --kubernetes-version=v1.26 --container-runtime docker --gpus all
# minikube start -p kubeflow --cpus 4 --memory 8096 --kubernetes-version=v1.26 --driver=docker
```

```bash
minikube profile list
```

```bash
kubectl get pods -A   # 실행 확인
```

## Kustomize 설치 (v5.0.3)

<!-- ```bash
wget https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv5.0.3/kustomize_v5.0.3_linux_amd64.tar.gz
``` -->

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash -s -- 5.0.3
sudo mv kustomize /usr/local/bin/
kustomize version
```

## Kubeflow menifests 구성 및 배포 (v1.8)

```bash
cd ~
mkdir kubeflow
cd kubeflow
```

```bash
git clone -b v1.8.0 https://github.com/kubeflow/manifests.git
cd manifests
while ! kustomize build example | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```

설치하는 동안 기다린다.

```bash
kubectl get pods -A
kubectl get namespaces
```

기다리다 보면 모든 Pod가 정상 실행되는 것을 확인할 수 있다.

## 대시보드 접속

```bash
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

### 접속

[http://localhost:8080](http://localhost:8080)

### 기본 사용자

- Email: user@example.com
- Password: 12341234

## User 생성 (* Pods `Running` 상태 확인 후)

```bash
nano profile.yaml
# code profile.yaml
```

아래 내용 복사, 수정

```yaml
apiVersion: kubeflow.org/v1
kind: Profile
metadata:
  name: sungjin    ## Namespace 이름 수정
spec:
  owner:
    kind: User
    name: sodyn99@gmail.com    ## Email 수정

  ## plugins extend the functionality of the profile
  ## https://github.com/kubeflow/kubeflow/tree/master/components/profile-controller#plugins
  plugins: []

  ## optionally create a ResourceQuota for the profile
  ## https://github.com/kubeflow/kubeflow/tree/master/components/profile-controller#resourcequotaspec
  ## https://kubernetes.io/docs/reference/kubernetes-api/policy-resources/resource-quota-v1/#ResourceQuotaSpec
  resourceQuotaSpec:
    hard:
      cpu: "4"
      memory: "16Gi"
      reqeusts.nvidia.com/gpu: "1"
      persistentvolumeclaims: "10"
      requests.storage: "100Gi"
```

```bash
kubectl apply -f profile.yaml
```

### Profile 확인

```bash
kubectl get profiles
kubectl describe profile <PROFILE_NAME>
```

### Profile 삭제

```bash
kubectl delete profile <PROFILE_NAME>
```

### Profile 적용

```bash
kubectl delete deployments.apps dex -n auth
```


### 비밀번호 설정

```bash
nano manifests/common/dex/overlays/oauth2-proxy/config-map.yaml
# code manifests/common/dex/overlays/oauth2-proxy/config-map.yaml
```

```yaml
...
  staticPasswords:
  - email: user@example.com   # Email 수정
    hash: $2y$12$4K/VkmDd1q1Orb3xAt82zu8gk7Ad6ReFR4LCP9UeYE90NLiN9Df72 # 해시된 비밀번호 수정
    username: user    # User 이름 수정
    userID: "15841185641784"    # User ID 수정
    ...
  staticClients:
    redirectURIs: ["/authservice/oidc/callback"]    # 수정
```


[Bcrypt generator](https://bcrypt-generator.com/)를 이용해 비밀번호 해시

```bash
kustomize build ./manifests/common/dex/overlays/oauth2-proxy | kubectl apply -f -
```

확인

```bash
...
configmap/dex configured
...
```


### 접속

```bash
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

## 기타


### Pods, PVC

- pod 확인

  ```bash
  kubectl get pods -n <PROFILE_NAME>
  ```
- pvc 확인, 삭제

  ```bash
  kubectl get pvc -n <PROFILE_NAME>
  kubectl delete persistentvolumeclaim <PVC_NAME> -n <PROFILE_NAME>
  ```

### Minikube

```bash
minikube profile list
minikube stop -p <Node_Name>
minikube delete -p <Node_Name>
```

### Docker GPU

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

```bash
docker run --rm --gpus all ubuntu:20.04 nvidia-smi
```

```python
import tensorflow as tf
tf.test.is_gpu_available()
```

