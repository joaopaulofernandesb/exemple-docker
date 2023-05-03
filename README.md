
# Exemplo de alta performance com Docker e Kubernetes em Node.js

Uma breve descrição sobre o que esse projeto faz e para quem ele é

Helpful Assistant
Exemplo de alta performance com Docker, Docker Compose e Kubernetes escalável em Node.js
Este é um exemplo de como criar um aplicativo Node.js simples que retorna "Hello World!" quando acessado, usando Docker, Docker Compose e Kubernetes para torná-lo escalável.

# Pré-requisitos

    1- Docker instalado
    2- Docker Compose instalado
    3- Kubernetes instalado

# Passo a passo
  1 - Crie um novo diretório e crie um arquivo chamado **index.js** com o seguinte conteúdo:
```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200);
  res.end('Hello World!');
});

server.listen(3000);
```

2 - Crie um arquivo Dockerfile para construir uma imagem Docker para o nosso aplicativo Node.js:

```dockerfile

FROM node:14-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

CMD ["node", "index.js"]
```
Este Dockerfile usa a imagem **node:14-alpine** como base, copia os arquivos do aplicativo para o diretório de trabalho **/app**, instala as dependências do Node.js e, em seguida, inicia o aplicativo com o comando **node index.js**.


3 - Crie um arquivo docker-compose.yml para definir um serviço Docker Compose para o nosso aplicativo Node.js:

```yaml
version: '3'

services:
  app:
    build: .
    ports:
      - "3000:3000"
```
Este arquivo **docker-compose.yml** define um serviço chamado **app** que usa a imagem Docker construída a partir do nosso Dockerfile e expõe a porta 3000.

4 - Implantar o aplicativo Node.js em um cluster Kubernetes. Primeiro, crie um arquivo **deployment.yaml** para definir um objeto Deployment do Kubernetes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
        - name: app
          image: <your-docker-registry>/node-app:latest
          ports:
            - containerPort: 3000
```

Este arquivo deployment.yaml define um objeto Deployment chamado app que cria três réplicas do nosso aplicativo Node.js. Ele usa um seletor para garantir que todas as réplicas tenham a mesma etiqueta app: app. A especificação do modelo define um contêiner que usa a imagem Docker que construímos anteriormente e expõe a porta 3000.

5 - Crie um arquivo service.yaml para definir um objeto Service do Kubernetes:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  selector:
    app: app
  ports:
    - name: http
      port: 80
      targetPort: 3000
  type: LoadBalancer
```
Este arquivo service.yaml define um objeto Service chamado app que usa o seletor app: app para rotear o tráfego para as réplicas do nosso aplicativo Node.js. Ele expõe a porta 80 e direciona o tráfego para a porta 3000 do contêiner. O tipo de serviço é LoadBalancer, o que significa que o Kubernetes provisionará um balanceador de carga externo para rotear o tráfego para as réplicas do nosso aplicativo.

6 - Implantar o aplicativo Node.js no Kubernetes executando os seguintes comandos:

```bash
# Fazer o build da imagem Docker
docker build -t <your-docker-registry>/node-app:latest .

# Fazer o push da imagem Docker para o registro
docker push <your-docker-registry>/node-app:latest

# Aplicar os arquivos de configuração do Kubernetes
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```
Agora, nosso aplicativo Node.js está implantado em um cluster Kubernetes e é escalável. Podemos aumentar ou diminuir o número de réplicas do aplicativo ajustando o valor replicas na especificação do objeto Deployment. O Kubernetes cuidará automaticamente de distribuir o tráfego entre as réplicas do aplicativo.
