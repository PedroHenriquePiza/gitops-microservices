# README — GitOps na prática: Online Boutique (Rancher Desktop + ArgoCD)

Este README descreve **passo a passo** para executar o projeto *Online Boutique* localmente usando **Rancher Desktop (Kubernetes)** e **ArgoCD** com GitOps. Os comandos abaixo são pensados para **PowerShell no Windows**.

---

## Sumário
- Pré-requisitos
- 1 — Instalar e configurar Rancher Desktop
- 2 — Instalar Git e (opcional) Docker CLI
- 3 — Preparar o repositório GitHub (manifests)
- 4 — Instalar ArgoCD no cluster
- 5 — Acessar o ArgoCD (port-forward + senha)
- 6 — Criar a aplicação no ArgoCD (UI e CLI)
- 7 — Acessar o frontend (port-forward)
- 8 — Testar GitOps (edição e sincronização)
- Dicas e resolução de problemas

---

## Pré-requisitos
- Windows 10/11 (com privilégios administrativos)
- PowerShell (executar como Administrador quando necessário)
- Conexão com a internet
- Conta no GitHub

---

## 1 — Instalar e configurar Rancher Desktop
1. Baixe e instale: https://rancherdesktop.io/  
2. Abra Rancher Desktop → **Settings → Kubernetes** → habilite **Enable Kubernetes**.  
3. Escolha `containerd` (ou `dockerd` se preferir Docker CLI). Aguarde até o cluster iniciar.  
4. Verifique no PowerShell:
```powershell
kubectl get nodes
```

---

## 2 — Instalar Git e (opcional) Docker CLI
Instalar Git via winget:
```powershell
winget install --id Git.Git -e --source winget
git --version
```
(Se desejar Docker CLI)
```powershell
winget install Docker.DockerCLI
```

---

## 3 — Preparar o repositório GitHub (manifests)
1. Faça fork do repositório oficial (opcional):  
   `https://github.com/GoogleCloudPlatform/microservices-demo`
2. Crie um repositório público seu, por exemplo `gitops-microservices`.
3. Dentro do repositório crie a estrutura:
```
gitops-microservices/
└── k8s/
    └── online-boutique.yaml
```
4. Copie o conteúdo do manifest original (release/kubernetes-manifests.yaml) para `k8s/online-boutique.yaml` e faça commit & push.

---

## 4 — Instalar ArgoCD no cluster
> Observação: se acessar https://raw.githubusercontent.com devolver `429 Too Many Requests`, use a alternativa de download local.

Criar namespace e aplicar manifest:
```powershell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/install.yaml
```
Se receber erro 429:
- Baixe `install.yaml` via navegador (https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/install.yaml) e então:
```powershell
kubectl apply -n argocd -f C:\caminho\para\install.yaml
```
Verifique:
```powershell
kubectl get pods -n argocd
```

---

## 5 — Acessar o ArgoCD (port-forward + senha)
1. No PowerShell (manter janela aberta):
```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
2. Abra no navegador:
```
https://localhost:8080
```
3. Obter senha do `admin` (PowerShell — decodifica Base64):
```powershell
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | %{ [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```
Usuário: `admin`  
Senha: valor retornado pelo comando acima.

---

## 6 — Criar a aplicação no ArgoCD

### Opção A — Via interface web (GUI)
1. No ArgoCD clique em **NEW APP**.
2. Preencha:
- **Application Name:** `online-boutique`  
- **Project:** `default`  
- **Repository URL:** `https://github.com/<seu-usuario>/gitops-microservices.git`  
- **Revision:** `HEAD`  
- **Path:** `k8s`  
- **Cluster:** `https://kubernetes.default.svc`  
- **Namespace:** `default`  
3. Clique em **Create**, depois em **SYNC** → **SYNCHRONIZE**.

### Opção B — Via CLI `argocd`
> O cliente `argocd` pode ser instalado separadamente. Exemplo de uso:
```powershell
argocd login localhost:8080 --username admin --password "SUA_SENHA" --insecure

argocd app create online-boutique `
  --repo https://github.com/<seu-usuario>/gitops-microservices.git `
  --path k8s `
  --dest-server https://kubernetes.default.svc `
  --dest-namespace default

argocd app sync online-boutique
```

Verifique pods:
```powershell
kubectl get pods
```

---

## 7 — Acessar o frontend da aplicação
1. Lista services:
```powershell
kubectl get svc
```
2. Normalmente o service do frontend se chama `frontend` (ou `frontend-external`). Faça port-forward:
```powershell
kubectl port-forward svc/frontend 8081:80
# ou se tiver frontend-external:
kubectl port-forward svc/frontend-external 8081:80
```
3. Abra no navegador:
```
http://localhost:8081
```

---

## 8 — Testar GitOps (edição e sincronização)
1. Edite `k8s/online-boutique.yaml` no seu repositório (ex.: altere `replicas: 1` para `replicas: 2` em algum Deployment).
2. Commit & push.
3. No ArgoCD você verá o app como `OutOfSync`. Clique em **Sync** para aplicar a mudança, ou habilite sincronização automática (Auto-Sync) no Application settings para aplicar automaticamente.

---

## Dicas e resolução de problemas
- **429 Too Many Requests** ao usar raw.githubusercontent: baixe o `install.yaml` manualmente e aplique localmente.  
- **Erro ao decodificar base64 no PowerShell**: use o comando com `[System.Convert]::FromBase64String()` (conforme seção 5).  
- **Port-forward não conecta**: verifique se o nome do service está correto (`kubectl get svc`) e se o pod correspondente está `Running`.  
- **Ver logs de um pod**:
```powershell
kubectl logs pod/<nome-do-pod>
# ou para ver rollout/descrição:
kubectl describe pod <nome-do-pod>
```
- **Ver recursos ArgoCD via CLI**:
```powershell
argocd app list
argocd app get online-boutique
```

---

## Comandos úteis (resumo)
```powershell
kubectl get nodes
kubectl get pods -n argocd
kubectl apply -n argocd -f C:\caminho\install.yaml
kubectl port-forward svc/argocd-server -n argocd 8080:443
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | %{ [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
argocd app create online-boutique --repo https://github.com/<seu-usuario>/gitops-microservices.git --path k8s --dest-server https://kubernetes.default.svc --dest-namespace default
argocd app sync online-boutique
kubectl port-forward svc/frontend 8081:80
```
