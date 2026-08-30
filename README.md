# oprimoguslab-gitops


## Visão Geral

`oprimoguslab-gitops` é um repositório GitOps que gerencia a infraestrutura e os aplicativos de produção do meu cluster Kubernetes / homelab `oprimoguslab`.  
O repositório está organizado de forma a separar claramente **infraestrutura** (operators, CRDs, etc.) e **aplicativos** (deployments, services, etc.) utilizando **ArgoCD** e **ApplicationSets** para automação de sincronização.
