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

``` bash
minikube start --cpus 4 --memory 8096 --kubernetes-version=v1.26
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
kubectl get namepaces
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

## User 생성

```bash
nano profile.yaml
```

아래 내용 복사, 수정

```yaml
apiVersion: kubeflow.org/v1
kind: Profile
metadata:
  name: sungjin    ## namespace 이름 수정
spec:
  owner:
    kind: User
    name: ssjj3552@gmail.com    ## email 수정

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
kubectl describe profile MY_PROFILE_NAME
```

### Profile 삭제

```bash
kubectl delete profile MY_PROFILE_NAME
```