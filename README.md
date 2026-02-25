# Infraestrutura MVP no Minikube

## Estrutura de manifests

```
infra/
  k8s/
    kustomization.yaml
    platform/
    apps/
    observability/
    overlays/local/
```

## Pre-requisitos

- Docker Desktop ativo
- Minikube instalado
- kubectl instalado

## 1. Subir o cluster

```bash
minikube start --cpus=4 --memory=8192
kubectl config use-context minikube
```

## 2. Apontar o Docker para o daemon do Minikube

### Linux/macOS (bash)

```bash
eval $(minikube docker-env)
```

### Windows PowerShell

```powershell
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
```

## 3. Buildar as imagens locais

Execute os comandos a partir da raiz do repositorio:

```bash
docker build -t agrosolutions/identity-service:local -f AgroSolutionsIdentity/Dockerfile AgroSolutionsIdentity
docker build -t agrosolutions/properties-service:local -f AgroSolutionsProperties/Dockerfile AgroSolutionsProperties
docker build -t agrosolutions/ingestion-service:local -f AgroSolutionsIngestion/Dockerfile AgroSolutionsIngestion
docker build -t agrosolutions/analytics-service:local -f AgroSolutionAnalytics/Dockerfile AgroSolutionAnalytics
docker build -t agrosolutions/analytics-worker:local -f AgroSolutionAnalytics/Dockerfile.Worker AgroSolutionAnalytics
```

## 4. Aplicar os manifests no Kubernetes

```bash
kubectl apply -k infra/k8s/overlays/local
```

## 5. Validar a subida dos componentes

```bash
kubectl get ns
kubectl get pods -n agrosolutions-app
kubectl get pods -n agrosolutions-observability
kubectl get svc -n agrosolutions-app
kubectl get svc -n agrosolutions-observability
```

Opcional (acompanhar ate ficar pronto):

```bash
kubectl get pods -n agrosolutions-app -w
```

## 6. Acessar APIs e observabilidade

Abra um terminal por comando de port-forward:

```bash
kubectl -n agrosolutions-app port-forward svc/identity-service 5001:8080
kubectl -n agrosolutions-app port-forward svc/properties-service 5002:8080
kubectl -n agrosolutions-app port-forward svc/ingestion-service 5003:8080
kubectl -n agrosolutions-app port-forward svc/analytics-service 5004:8080
kubectl -n agrosolutions-observability port-forward svc/prometheus 9090:9090
kubectl -n agrosolutions-observability port-forward svc/grafana 3001:3000
```

URLs:

- Identity Swagger: `http://localhost:5001/swagger`
- Properties Swagger: `http://localhost:5002/swagger`
- Ingestion Swagger: `http://localhost:5003/swagger`
- Analytics Swagger: `http://localhost:5004/swagger`
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3001` (usuario `admin`, senha `admin`)

## 7. Comandos de diagnostico

Logs de um deployment:

```bash
kubectl logs -n agrosolutions-app deploy/identity-service --tail=200
```

Descricao detalhada de pod:

```bash
kubectl describe pod <pod-name> -n agrosolutions-app
```

Reiniciar um servico:

```bash
kubectl rollout restart deployment/identity-service -n agrosolutions-app
```

## 8. Limpeza do ambiente

Remover tudo do Kubernetes:

```bash
kubectl delete -k infra/k8s/overlays/local
```

Excluir cluster Minikube:

```bash
minikube delete
```
