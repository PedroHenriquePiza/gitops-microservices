# README — GitOps na prática: Online Boutique (Rancher Desktop + ArgoCD)

Este README descreve **passo a passo** para executar o projeto *Online Boutique* localmente usando **Rancher Desktop (Kubernetes)** e **ArgoCD** com GitOps. Os comandos abaixo são pensados para **PowerShell no Windows**.

---

## Pré-requisitos
- 1 — Instalar e configurar Rancher Desktop
- 2 — Instalar Git e  Docker CLI
- 3 — Preparar o repositório GitHub 
- 4 — Instalar ArgoCD no cluster
- 5 — Acessar o ArgoCD
- 6 — Criar a aplicação no ArgoCD 
- 7 — Acessar o frontend 
- 8 — Testar GitOps

---

## Pré-requisitos
- Windows 10/11 
- PowerShell (executar como Administrador quando necessário)
- Conexão com a internet
- Conta no GitHub

---

## 1 — Instalar e configurar Rancher Desktop
1. Baixe e instale: https://rancherdesktop.io/    
2. Escolha `containerd` . Aguarde até o cluster iniciar.  
3. Verifique no PowerShell:
```powershell
kubectl get nodes
```

---

## 2 — Instalar Git e Docker CLI
Instalar Git via winget:
```powershell
winget install --id Git.Git -e --source winget
git --version
```

```powershell
winget install Docker.DockerCLI
```

---

## 3 — Preparar o repositório GitHub 
1. Faça fork do repositório oficial :  
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

## 5 — Acessar o ArgoCD 
1. No PowerShell (manter janela aberta):
```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
2. Abra no navegador:
```
https://localhost:8080
```
3. Obter senha do `admin` :
```powershell
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | %{ [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```
Usuário: `admin`  
Senha: valor retornado.

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
2. Normalmente o service do frontend se chama `frontend` . Faça port-forward:
```powershell
kubectl port-forward svc/frontend 8081:80
# ou se tiver frontend-external:
kubectl port-forward svc/frontend-external 8081:80
```
3. Abra no navegador:
```
http://localhost:8081
```

