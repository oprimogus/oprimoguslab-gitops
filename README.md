# oprimoguslab-gitops

## Visão Geral  
`oprimoguslab-gitops` é um repositório GitOps que gerencia a infraestrutura e os aplicativos de produção do cluster Kubernetes **oprimoguslab** (homelab).  
Ele segue a arquitetura **ArgoCD + ApplicationSets** para sincronização automática e utiliza **Talos** como sistema operacional do cluster.

---

## Estrutura de Diretórios

```
├── bootstrap.sh                  # Instala ArgoCD e aplica os root‑apps
├── cluster                       # Configurações do Talos (controlplane, workers, secrets)
│   ├── controlplane.yaml         # Arquivo de configuração do controlplane
│   ├── worker.yaml               # Arquivo de configuração dos workers
│   ├── secrets.yaml              # Certificados PKI e chaves criptografadas
│   ├── talosconfig               # Talos config usado para join dos nós
│   └── config/                   # Configurações adicionais (volume, kubelet, etc.)
├── k8s
│   ├── argocd                    # Manifests ArgoCD (projects, applicationsets, root‑app)
│   │   ├── app-projects
│   │   │   ├── apps.yaml
│   │   │   ├── infrastructure.yaml
│   │   │   └── operators.yaml
│   │   ├── applicationsets
│   │   │   ├── apps.yaml
│   │   │   ├── infrastructure.yaml
│   │   │   └── operators.yaml
│   │   └── bootstrap
│   │       ├── root-app-projects.yaml
│   │       └── root-app.yaml
│   ├── apps                      # Helm/HelmCharts dos aplicativos de produção
│   ├── infrastructure            # Configurações de infraestrutura (CRDs, custom resources)
│   └── operators                 # Operadores e CRDs
└── README.md
```

> **Obs.** Todos os manifests de aplicação estão no diretório `k8s/apps` e são referenciados pelos *ApplicationSets*.

---

## Configuração do Cluster Talos

O Talos utiliza YAML de alta abstração para declarar a stack do cluster.

| Arquivo | Função |
|---------|--------|
| `cluster/controlplane.yaml` | Definição do nó de controle (token, PKI, kubelet, instalação, etc.) |
| `cluster/worker.yaml` | Definição dos nós de trabalho |
| `cluster/secrets.yaml` | Certificados PKI (`ca.crt/key`), `k8s` certs, `talos` certs e outras chaves criptografadas (base64). |
| `cluster/config/` | Configurações específicas, como volumes persistentes (`volume.yaml`) e kubelet (`kubelet-cert.yaml`). |

### Criando o cluster

```bash
# 1. Instale a CLI do Talos
curl -fsSL https://talos.dev/install.sh | sh

# 2. Conecte o controlplane
talosctl --talosconfig cluster/talosconfig apply-config \
    --nodes <IP_CONTROL> cluster/controlplane.yaml

# 3. Adicione workers
talosctl --talosconfig cluster/talosconfig apply-config \
    --nodes <IP_WORKER> cluster/worker.yaml
```

> Os arquivos `talosconfig` e `secrets.yaml` são gerados pela CLI ou pelo script de bootstrap. Consulte a documentação do Talos para detalhes sobre tokens e chaves.

---

## ArgoCD e Bootstrap

`bootstrap.sh` realiza os seguintes passos:

1. Cria o namespace **argocd**.  
2. Aplica o manifest oficial de instalação (`install.yaml`).  
3. Aguarda o deployment do servidor ArgoCD ficar disponível.  
4. Recupera a senha inicial (`argocd-initial-admin-secret`).  
5. Aplica os *root‑apps* que informam ao ArgoCD onde encontrar os projetos e application‑sets.

Os root‑apps estão em `k8s/argocd/bootstrap/`. Eles apontam para a branch **talos** e para os caminhos corretos dentro do repositório.

---

## ApplicationSets

Cada *ApplicationSet* utiliza um *generator* Git para escanear os diretórios adequados:

| ApplicationSet | Generator | Target |
|----------------|-----------|--------|
| `apps` | `git.files` | `k8s/apps/*.yaml` |
| `infrastructure` | `git.directories` | `k8s/infrastructure/*` |
| `operators` | `git.files` | `k8s/operators/*` |

Os projetos ArgoCD correspondentes (`apps`, `infrastructure`, `operators`) definem permissões de acesso a namespaces e recursos.

---

## Como Usar

1. **Bootstrap** – Execute `./bootstrap.sh` (necessita de `kubectl` configurado para seu cluster).  
   Isso instalará ArgoCD e criará os root‑apps.  
2. **Sincronizar** – No dashboard do ArgoCD, cada *Application* aparecerá e será sincronizado automaticamente graças à política `automated`.  
3. **Deploy** – Adicione ou edite arquivos YAML em `k8s/apps`, `k8s/infrastructure` ou `k8s/operators`; o ArgoCD detectará e aplicará automaticamente.  
4. **Manter** – Faça merges na branch **talos** apenas quando for necessário atualizar a stack do cluster.

---

## Próximos Passos

* [ ] Documentar o processo de geração de certificados PKI.  
* [ ] Explicar a política de *secretbox encryption* usada em `secrets.yaml`.  
* [ ] Adicionar exemplos de deployment de um novo aplicativo via Helm.  

---  

*Autor:* **Oprimogus** – 2026