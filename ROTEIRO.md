# Roteiro de explicacao da infraestrutura

## Objetivo da conversa

Este conjunto de arquivos foi criado para colocar o MVP no Kubernetes de forma simples, local e funcional.
A ideia e ter tudo que o projeto precisa para subir: aplicacoes, banco, mensageria e monitoramento.

## Como contar a estrutura em 1 minuto

Dentro de `infra/k8s`, a organizacao foi separada em 4 blocos:

- `platform/`: base da plataforma (namespaces, secrets/configs, banco, fila e persistencia).
- `apps/`: deploy dos microsservicos da aplicacao.
- `observability/`: Prometheus e Grafana para metricas e dashboards.
- `overlays/local/`: ajuste de imagens para rodar local no Minikube.

O arquivo `infra/k8s/kustomization.yaml` junta tudo isso em um unico deploy.

## Explicacao arquivo por arquivo (visao pratica)

### 1) Platform

- `namespaces.yaml`
  - Cria dois namespaces:
    - `agrosolutions-app`: onde rodam os microsservicos e dependencias.
    - `agrosolutions-observability`: onde rodam Prometheus e Grafana.

- `app-configmap.yaml`
  - Guarda configuracoes nao sensiveis, como ambiente da aplicacao e porta HTTP.

- `app-secrets.yaml`
  - Guarda dados sensiveis:
    - strings de conexao
    - credenciais do RabbitMQ
    - configuracao JWT
    - usuario/senha do Grafana

- `postgres-storage.yaml`, `mongodb-storage.yaml`, `rabbitmq-storage.yaml`, `prometheus-storage.yaml`, `grafana-storage.yaml`
  - Criam PV/PVC para manter dados mesmo se pod reiniciar.

- `postgres.yaml`, `mongodb.yaml`, `rabbitmq.yaml`
  - Sobem os servicos de infraestrutura com:
    - Deployment
    - Service interno
    - probes de saude
    - requests/limits de CPU e memoria

### 2) Apps

Arquivos:

- `identity-service.yaml`
- `properties-service.yaml`
- `ingestion-service.yaml`
- `analytics-service.yaml`
- `analytics-worker.yaml`

Cada arquivo define o Deployment (e Service nas APIs), com:

- imagem Docker
- variaveis vindas de ConfigMap/Secret
- probes de liveness/readiness
- recursos minimos e limites

Resumo de dependencias:

- Identity, Properties e Analytics API usam Postgres.
- Ingestion e Analytics (API/Worker) usam MongoDB.
- Ingestion e Worker usam RabbitMQ.

### 3) Observability

- `prometheus-configmap.yaml`
  - Define os targets de scrape (`/metrics`) dos microsservicos.

- `prometheus.yaml`
  - Deployment e Service do Prometheus com persistencia.

- `grafana-configmap.yaml`
  - Provisiona datasource do Prometheus e provider de dashboard.

- `grafana-dashboard-configmap.yaml`
  - Inclui dashboard inicial para acompanhar API request rate, CPU e memoria.

- `grafana.yaml`
  - Deployment e Service do Grafana com persistencia.

### 4) Overlay local

- `overlays/local/kustomization.yaml`
  - Troca as imagens para tags locais (`agrosolutions/...:local`), ideal para Minikube.

## Pipeline CI/CD

Arquivo:

- `.github/workflows/cicd-k8s.yml`

Fluxo:

1. Faz build das 5 imagens.
2. Publica no GHCR.
3. Aplica manifests no cluster com `kubectl apply -k infra/k8s`.
4. Atualiza os deployments para usar imagem do commit (`github.sha`).

## Como explicar a execucao local (passo rapido)

1. Sobe Minikube.
2. Aponta Docker para o daemon do Minikube.
3. Builda imagens locais.
4. Aplica `kubectl apply -k infra/k8s/overlays/local`.
5. Valida pods e services.
6. Faz port-forward para APIs, Prometheus e Grafana.

O guia de comandos ja esta em:

- `infra/README.md`

## Sugestão encerramento

Com isso, o MVP passa a ter um ambiente padrao de execucao em Kubernetes, com dependencias internas, monitoramento e caminho de entrega continua.
Para evoluir depois, os proximos passos naturais seriam ingress, TLS e separacao de secrets por ambiente.

