# 🚀 CI/CD e GitOps na Prática com FastAPI, Docker Hub e ArgoCD

> Repositório responsável pelos **manifests Kubernetes** utilizados no **deploy automatizado** da aplicação **FastAPI** via **ArgoCD** seguindo o modelo **GitOps**.

<br>

<p align="center">
  <img src="https://skillicons.dev/icons?i=kubernetes,github" />
  <img src="https://argo-cd.readthedocs.io/en/stable/assets/logo.png" width="48" height="48" alt="ArgoCD"/>
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/yaml/yaml-original.svg" width="48" height="48" alt="YAML"/>
</p>

<br>

## 🧭 Introdução

Este repositório faz parte do projeto de **CI/CD e GitOps com FastAPI**, onde o deploy é **totalmente automatizado** por meio do **ArgoCD**.  

Aqui ficam armazenados os arquivos **YAML** responsáveis por descrever os recursos Kubernetes da aplicação, e são automaticamente atualizados via **GitHub Actions** do repositório `hello-app`.

<br>

## 🎯 Objetivo

O objetivo é manter a infraestrutura da aplicação versionada e sincronizada com o cluster Kubernetes.  
Toda alteração feita aqui (como atualização de imagem ou configuração de pods) é **detectada e aplicada automaticamente** pelo **ArgoCD**.

```bash
+------------------+       +------------------+       +--------------------+       +-------------------+
|    hello-app     |       |   Docker Hub     |       |   hello-manifests  |       |     ArgoCD        |
| (FastAPI + CI/CD)| ----> | (Container Repo) | ----> | (K8s Manifests Git)| ----> | (Sync no Cluster) |
+------------------+       +------------------+       +--------------------+       +-------------------+
```

<br>

## ⚙️ Pré-requisitos

Antes de começar, garanta que as ferramentas abaixo estejam instaladas e configuradas:

| Ferramenta | Função | Verificação |
|-------------|--------|-------------|
| **Git** | Versionamento e controle de código | `git --version` |
| **GitHub** | Hospedagem dos repositórios | Conta criada |
| **Rancher Desktop** | Kubernetes local | `kubectl get nodes` |
| **ArgoCD** | Entrega contínua GitOps | Instalado no cluster |

<br>

## 🧩 Estrutura dos Repositórios

Este projeto utiliza **dois repositórios GitHub**:

### 1️⃣ Repositório [`hello-app`](https://github.com/pedro-albertini/hello-app)
Contém:
- Aplicação FastAPI [`main.py`](https://github.com/pedro-albertini/hello-app/blob/main/main.py)
- [`Dockerfile`](https://github.com/pedro-albertini/hello-app/blob/main/Dockerfile)
- Workflow [`.github/workflows/main.yml`](https://github.com/pedro-albertini/hello-app/blob/main/.github/workflows/main.yaml)

Responsável pela primeira etapa do processo:
- Buildar e publicar imagens no Docker Hub  
- Atualizar o repositório de manifests (`hello-manifests`)

### 2️⃣ Repositório [`hello-manifests`](https://github.com/pedro-albertini/hello-manifests)
Contém os arquivos Kubernetes:
- [`deployment.yaml`](https://github.com/pedro-albertini/hello-manifests/blob/main/deployment.yaml)
- [`service.yaml`](https://github.com/pedro-albertini/hello-manifests/blob/main/service.yaml)

Responsável por:
- Armazenar os manifests observados pelo ArgoCD  
- Garantir o modelo GitOps, onde o **Git é a fonte da verdade**

<br>


## 🧱 Etapa 1 – Criar os manifests do Kubernetes

Crie um novo repositório chamado por exemplo de hello-manifests e adicione os arquivos de manifesto do kubernetes:

deployment.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
        - name: hello-app
          image: <Seu Docker Hub>/hello-app:latest
          ports:
            - containerPort: 8000

```

service.yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: hello-app-service
spec:
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8000
  type: ClusterIP

```
<br>

## ☸️ Etapa 2 – Configurar o ArgoCD

Primeiro no terminal, instale o ArgoCD:

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verifique se todos os pods estão rodando:

```
kubectl get pods -n argocd
```

| <img width="1917" height="319" alt="image" src="https://github.com/user-attachments/assets/863741d9-3c78-4c3b-b327-e58afb4d4071" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Pod ArgoCD Rodando* |


Crie o port-forward para acessa-lo:

```
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Acesse no navegador:
🔗 http://localhost:8081

E você verá uma página assim:

| <img width="1914" height="540" alt="image" src="https://github.com/user-attachments/assets/e0b237f1-3ba2-4c50-9e05-308189c35541" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Painel ArgoCD* |

Credenciais padrão:

- Usuário: admin

- Senha: (use o comando abaixo para descobrir)

```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo
```


##  🌐 Etapa 3 - Criar o app no ArgoCD

No painel do ArgoCD, clique em NEW APP

Preencha os campos de acordo com a tabela a seguir:

| Campo                | Valor                                                                                                            |
| -------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Application Name** | hello-app                                                                                                        |
| **Project**          | default                                                                                                          |
| **Repository URL**   | [https://github.com/pedro-albertini/hello-manifests.git](https://github.com/pedro-albertini/hello-manifests.git) |
| **Revision**         | HEAD                                                                                                             |
| **Path**             | `.`                                                                                                              |
| **Cluster URL**      | [https://kubernetes.default.svc](https://kubernetes.default.svc)                                                 |
| **Namespace**        | default                                                                                                          |

<br>

Ficando assim o preenchimento dos campos no ArgoCD:

| <img width="1917" height="867" alt="image" src="https://github.com/user-attachments/assets/5e2aa07c-ed54-4449-a37b-b3f9d06329fe" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Configuração Aplicação ArgoCD* |

| <img width="1918" height="868" alt="image" src="https://github.com/user-attachments/assets/06a9380b-1986-4a6a-a53e-848ae6964273" />|
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Configuração Aplicação ArgoCD* |

| <img width="1915" height="862" alt="image" src="https://github.com/user-attachments/assets/5758b4e6-6634-46f3-a231-0c8bd4cf50ee" />|
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Configuração Aplicação ArgoCD* |

- Clique em Create

- Depois, clique em SYNC → SYNCHRONIZE

| <img width="1917" height="868" alt="image" src="https://github.com/user-attachments/assets/799c8f09-3574-4560-b610-3437b4c63e17" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Aplicação Sincronizado* |

<br>

- Verifique se os pods/deployment da aplicação estão rodando:

```
kubectl get pods
```

| <img width="1819" height="81" alt="image" src="https://github.com/user-attachments/assets/6187682b-0af8-45bb-9cad-7e6e0b3e03cd" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Deployment Aplicação* |

<br>

## 🖥️ Etapa 4 – Acessar a aplicação

Verifique os pods:

```
kubectl get pods
```

Crie o port-forward:

```
kubectl port-forward svc/hello-app-service 8080:8080
```

Acesse:
🔗 http://localhost:8080

E você verá sia aplicação rodando:

| <img width="1914" height="974" alt="image" src="https://github.com/user-attachments/assets/3aa6f440-61cb-4b69-a8b5-871cf8150ef7" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Aplicação Rodando* |

<br>

## 🧪 Etapa 5 – Testar o deploy

Edite o arquivo `main.py` no repositório `hello-app` e altere o return:

```python
return {"message": "Hello Compass"}
```

Faça commit e push no repositório hello-app.

O GitHub Actions buildará uma nova imagem:

| <img width="1912" height="969" alt="image" src="https://github.com/user-attachments/assets/549d740f-4b3d-4587-a906-f9cae27262d4" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Build GitHub Actions* |

<br>

Publicará no Docker Hub:

| <img width="912" height="574" alt="image" src="https://github.com/user-attachments/assets/e175fe6b-d2f1-4e78-a289-3a37075d3fa0" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Imagem Docker Hub* |

<br>

E atualizará o repositório hello-manifests com a mesma tag que foi publicado no Docker Hub:

| <img width="1893" height="591" alt="image" src="https://github.com/user-attachments/assets/740f0c38-af86-4fa2-8905-f13636505c77" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Repositório Atualizado* |



O ArgoCD detectará a mudança e fará o deploy automaticamente.

Após a sincronização, atualize a página em http://localhost:8080 — a nova mensagem aparecerá!

| <img width="1912" height="967" alt="image" src="https://github.com/user-attachments/assets/a649ceaa-0393-48f9-9d0e-37ad09ba2462" /> |
|-------------------------------------------------------------------------------------------------------------------------|
| *Figura - Aplicação Atualizando* |

<br>

## 🧾 Conclusão

Este projeto demonstra, de forma prática, o funcionamento do ciclo completo de CI/CD e GitOps:
desde o desenvolvimento e build automatizado, até a entrega contínua via ArgoCD.

Com essa abordagem, toda a infraestrutura e o estado da aplicação ficam versionados no Git, garantindo rastreabilidade, segurança e velocidade nas entregas.

---
🧑‍💻 Desenvolvido por [Pedro Albertini Fernandes Pinto](https://github.com/pedro-albertini) 
Projeto prático do módulo **Automação CI/CD e GitOps com FastAPI e ArgoCD**
