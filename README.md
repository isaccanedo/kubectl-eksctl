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

### Verificar se o cluster foi criado

```
# Verificar nodes
kubectl get nodes

# Verificar se o contexto está configurado
kubectl config current-context
```

### Criar os arquivos YAML para sua aplicação

```
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
        image: isac/demo-auth:1.0
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
