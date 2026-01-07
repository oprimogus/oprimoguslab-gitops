# oprimoguslab-gitops


## Visão Geral

`oprimoguslab-gitops` é um repositório GitOps que gerencia a infraestrutura e os aplicativos de produção do meu cluster Kubernetes / homelab `oprimoguslab`.  
O repositório está organizado de forma a separar claramente **infraestrutura** (operators, CRDs, etc.) e **aplicativos** (deployments, services, etc.) utilizando **ArgoCD** e **ApplicationSets** para automação de sincronização.

## Estrutura do Repositório

```/dev/null/README.md#L30-60
k8s/
├─ apps/
│  ├─ charts/          # Helm charts para aplicações
│  │   ├─ minecraft/values.yaml
│  │   ├─ nextcloud/values.yaml
│  │   └─ …
│  └─ manifests/       # Manifests estáticos (se houver)
├─ infrastructure/
│  ├─ charts/
│  │   ├─ apps/          # Apps gerais que serão usados por varios apps independente de ambiente
│  │   │   ├─ hashicorp-vault/values.yaml
│  │   │   └─ zitadel/values.yaml
│  │   └─ operators/      # Operators
│  │       ├─ cert-manager/values.yaml
│  │       └─ …
│  └─ manifests/
└─ argocd/
    ├─ app-projects
    ├─ applicationsets
    └─ bootstrap/
        └─ root-app.yaml
```

## Como Usar

```/dev/null/README.md#L60-80
# Clone o repositório
git clone https://github.com/oprimogus/oprimoguslab-gitops.git
```

```/dev/null/README.md#L90-100
# Execute o script de bootstrap
# Ele instala o ArgoCD num cluster k8s
./bootstrap.sh
```

```/dev/null/README.md#L100-110
# Acesse o console do ArgoCD
argocd login <argocd-server> --username admin --password <token>
```

# Ou use port-forward para acessar o console do ArgoCD
kubectl port-forward svc/argocd-server -n argocd 8080:80

## Deploy de Aplicações

Cada aplicação é descrita por um arquivo `values.yaml` localizado em `k8s/apps/charts/<app>/values.yaml`.  
Quando um arquivo é adicionado ou alterado, o `ApplicationSet` correspondente detecta a mudança e cria ou atualiza o `Application` no ArgoCD.

### Adicionando uma Nova Aplicação

```/dev/null/README.md#L110-130
# Crie a pasta do aplicativo
mkdir -p k8s/apps/charts/<app>

# Crie o arquivo values.yaml
cat > k8s/apps/charts/<app>/values.yaml <<'EOF'
name: <app>
repoURL: https://<chart-repo>
targetRevision: <chart-version>
chart: <chart-name>
namespace: <app-namespace>
values: |
  <helm-values-yaml>
```

Commit e push o arquivo; o `ApplicationSet apps-charts` criará o `Application` automaticamente.

### Adicionando um Operator

```/dev/null/README.md#L130-150
# Crie a pasta do operador
mkdir -p k8s/infrastructure/charts/operators/<operator>

# Crie o arquivo values.yaml
cat > k8s/infrastructure/charts/operators/<operator>/values.yaml <<'EOF'
name: <operator>
repoURL: https://<operator-repo>
targetRevision: <chart-version>
chart: <chart-name>
namespace: <operator-namespace>
values: |
  <operator-specific-values>
EOF
```

Commit e push; o `ApplicationSet infra-charts` reconciliará o operador.


## Contato

- Mantainer: `oprimogus <oprimogus@github.com>`  
- Issue Tracker: https://github.com/oprimogus/oprimoguslab-gitops/issues