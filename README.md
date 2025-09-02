# kubectl-eksctl
:cloud: kubectl e eksctl são duas ferramentas diferentes para trabalhar com Kubernetes, especialmente no contexto da AWS

### kubectl
É a ferramenta de linha de comando oficial do Kubernetes que permite:
- Gerenciar recursos do cluster (pods, services, deployments, etc.)
- Executar comandos contra qualquer cluster Kubernetes
- Fazer deploy de aplicações
- Inspecionar e gerenciar recursos do cluster
- Visualizar logs e debugar aplicações

Funciona com qualquer distribuição Kubernetes (minikube, EKS, GKE, AKS, clusters on-premise, etc.).

### eksctl
É uma ferramenta específica para Amazon EKS (Elastic Kubernetes Service) que facilita:
- Criar e gerenciar clusters EKS na AWS
- Configurar node groups
- Gerenciar add-ons do EKS
- Simplificar operações complexas do EKS através de comandos simples


### Como instalar o eksctl para trabalhar com EKS?

```
No Linux/macOS
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

Verificar instalação
eksctl version
```

### Criar o cluster EKS

```
# Criar cluster EKS (demora cerca de 15-20 minutos)
eksctl create cluster \
  --name demo-cluster \
  --region us-east-1 \
  --nodegroup-name demo-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

### criar o node group

```
$ eksctl get nodegroup --cluster demo-cluster --region us-east-1
Error: No nodegroups found
~ $ eksctl create nodegroup \
>   --cluster demo-cluster \
>   --region us-east-1 \
>   --name demo-nodes-2 \
>   --node-type t3.medium \
>   --nodes 2 \
>   --nodes-min 1 \
>   --nodes-max 3 \
>   --managed
2025-09-02 01:49:36 [ℹ]  will use version 1.32 for new nodegroup(s) based on control plane version
2025-09-02 01:49:37 [ℹ]  nodegroup "demo-nodes-2" will use "" [AmazonLinux2023/1.32]
2025-09-02 01:49:37 [ℹ]  1 existing nodegroup(s) (demo-nodes) will be excluded
2025-09-02 01:49:37 [ℹ]  1 nodegroup (demo-nodes-2) was included (based on the include/exclude rules)
2025-09-02 01:49:37 [ℹ]  will create a CloudFormation stack for each of 1 managed nodegroups in cluster "demo-cluster"
2025-09-02 01:49:37 [ℹ]  
2 sequential tasks: { fix cluster compatibility, 1 task: { 1 task: { create managed nodegroup "demo-nodes-2" } } 
}
2025-09-02 01:49:37 [ℹ]  checking cluster stack for missing resources
2025-09-02 01:49:37 [ℹ]  cluster stack has all required resources
2025-09-02 01:49:37 [ℹ]  building managed nodegroup stack "eksctl-demo-cluster-nodegroup-demo-nodes-2"
2025-09-02 01:49:37 [ℹ]  deploying stack "eksctl-demo-cluster-nodegroup-demo-nodes-2"
2025-09-02 01:49:38 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-demo-nodes-2"
```

Este comando vai:

Criar 2 instâncias EC2 t3.medium
Configurar um node group gerenciado
Permitir escalar de 1 a 3 nodes


### Verificar se o cluster foi criado

```
# Verificar nodes
kubectl get nodes

# Verificar se o contexto está configurado
kubectl config current-context
```

### Criar os arquivos YAML para sua aplicação

```
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-auth-app
  labels:
    app: demo-auth
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-auth
  template:
    metadata:
      labels:
        app: demo-auth
    spec:
      containers:
      - name: demo-auth
        image: isaccanedo/demo-auth:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: demo-auth-service
spec:
  selector:
    app: demo-auth
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
EOF
```

### Aplicar os recursos no CloudShell

```
# Criar o arquivo deployment.yaml no CloudShell
cat > deployment.yaml << 'EOF'
# Cole o conteúdo do arquivo YAML acima aqui
EOF

# Aplicar os recursos
kubectl apply -f deployment.yaml
```

### Verificar o deploy e obter a URL pública

```
# Verificar pods
kubectl get pods

# Verificar services e obter a URL externa
kubectl get services demo-auth-service

# Aguardar o Load Balancer ficar disponível (pode demorar alguns minutos)
kubectl get services demo-auth-service -w
```

### Verificar clusters EKS existentes

```
# Listar todos os clusters EKS na região
eksctl get cluster

# Ou usando AWS CLI
aws eks list-clusters --region us-east-1
```

### Verificar status detalhado do cluster

```
# Status detalhado do cluster específico
eksctl get cluster --name demo-cluster --region us-east-1

# Ou detalhes completos via AWS CLI
aws eks describe-cluster --name demo-cluster --region us-east-1
```

### Verificar se o kubectl está configurado

```
# Ver contexto atual do kubectl
kubectl config current-context

# Listar todos os contextos disponíveis
kubectl config get-contexts

# Ver configuração completa
kubectl config view
```

### Testar conectividade com o cluster

```
# Verificar nodes do cluster
kubectl get nodes

# Ver informações detalhadas dos nodes
kubectl get nodes -o wide

# Verificar namespaces
kubectl get namespaces
```

### Verificar node groups

```
# Listar node groups
eksctl get nodegroup --cluster demo-cluster --region us-east-1

# Ou via AWS CLI
aws eks list-nodegroups --cluster-name demo-cluster --region us-east-1
```

### Monitorar criação em tempo real (se ainda estiver criando)

```
# Acompanhar logs de criação do cluster
eksctl utils describe-stacks --region us-east-1 --cluster demo-cluster

# Ver eventos do CloudFormation (onde o cluster é criado)
aws cloudformation describe-stacks --stack-name eksctl-demo-cluster-cluster --region us-east-1
```
```
Interpretação dos resultados:
✅ Cluster criado com sucesso quando:

eksctl get cluster mostra status "ACTIVE"
kubectl get nodes retorna seus nodes
kubectl config current-context mostra o contexto do EKS

⏳ Ainda criando quando:

eksctl get cluster mostra status "CREATING"
kubectl get nodes retorna erro de conexão
Processo pode demorar 15-20 minutos

❌ Erro na criação quando:

eksctl get cluster não mostra o cluster
Mensagens de erro nos comandos
Status "FAILED" ou similar
```

### Comando rápido para verificar tudo

```
echo "=== Verificando cluster EKS ==="
eksctl get cluster --region us-east-1
echo ""
echo "=== Verificando nodes ==="
kubectl get nodes
echo ""
echo "=== Verificando contexto kubectl ==="
kubectl config current-context
```

### Fazer o deploy da aplicação

```
# Aplicar os recursos no cluster
kubectl apply -f deployment.yaml
```

### Verificar o deploy

```
# Verificar se os pods foram criados
kubectl get pods

# Verificar se o service foi criado
kubectl get services

# Ver detalhes do deployment
kubectl get deployments
```

### Aguardar o Load Balancer ficar pronto

```
# Monitorar até aparecer o EXTERNAL-IP (pode demorar 2-5 minutos)
kubectl get services demo-auth-service -w
```

### Testar a aplicação
Quando o comando kubectl get services mostrar um EXTERNAL-IP (URL do Load Balancer da AWS), você poderá acessar sua aplicação:

```
# Pegar a URL externa
EXTERNAL_IP=$(kubectl get service demo-auth-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "Sua aplicação está disponível em: http://$EXTERNAL_IP"
```

### Custos e Limpeza
**Importante:** O EKS cobra por hora de cluster (~$0.10/hora) + instâncias EC2. Para evitar custos:

```
# Para deletar tudo quando não precisar mais
eksctl delete cluster --name demo-cluster --region us-east-1
```

### Resumo

```
1 - CloudShell: Ambiente Linux gratuito na AWS com ferramentas pré-instaladas
2 - EKS: Serviço gerenciado do Kubernetes na AWS
3 - Load Balancer: Criado automaticamente para expor sua aplicação na internet
4 - Sua aplicação: Roda em pods dentro do cluster, acessível via porta 8080

Tudo 100% na AWS, sem precisar instalar nada localmente!
```
