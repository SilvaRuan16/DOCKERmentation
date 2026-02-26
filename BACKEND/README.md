# Ferramentas Dockerizadas para Backend
Esta parte do guia mostrará como criar ambientes dockerizados para desenvolvimento, build e execução de suas aplicações, sendo as ferramentas mencionadas: Java  na versão 21.

## Java
Java é uma linguagem de programação orientada a objetos fortemente tipada, suas principais caracteristicas são sua segurança e robustez, desempenho, independente de plataforma e como mencionada anteriormente, orientação a objetos.

## Criar projeto Java
Acesse este site para criar o projeto: [Spring Initializr](https://start.spring.io/).
obs: Esta documentação foi pensada para a versão 21 do Java junto ao Maven e Spring Boot.

## Dockerfile para aplicações em Java e Desenvolvendo com VS Code (Extensão: Dev Containers) e Intelij (Docker)
Nesta etapa, mostraremos como usar o docker para desenvolvimento, build e execução.

Para prosseguir, será necessário baixar as extensões mostradas no titulo do capitulo.

### Configuração do Projeto
* #### 1. Crie uma pasta chamada `.devcontainer` na raiz do seu projeto.
* #### 2. Crie o arquivo `devcontainer.json` dentro do diretório .devcontainer.
* #### 3. Coloque este código abaixo dentro do arquivo `devcontainer.json`:
```
{
  "name": "Java 21 Dev Container",
  "image": "mcr.microsoft.com/devcontainers/java:21",
  "remoteUser": "root",
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",
  "mounts": [
    "source=${localEnv:HOME}/.m2,target=/home/vscode/.m2,type=bind"
  ],
  "customizations": {
    "vscode": {
      "extensions": [
        "vscjava.vscode-java-pack",
        "vscjava.vscode-spring-boot-dashboard",
        "vscjava.vscode-java-dependency"
      ],
      "settings": {
        "java.configuration.runtimes": [
          {
            "name": "JavaSE-21",
            "path": "/opt/java/openjdk",
            "default": true
          }
        ]
      }
    }
  }
}

```
### Como utilizar:
* Abra o projeto no Vs Code ou no Intelij.
* Possue várias formas de abrir o projeto do container na IDE, mas será mostrado somente um deles pois é o mais fácil:
  * Notificação: Se o arquivo .devcontainer estiver no projeto, a IDE costuma mostrar um balão no canto inferior direito dizendo: "Folder contains a Dev Container configuration file. Reopen to develop in a container?". Clique em `Reopen in Container`.
* Como saber se deu certo? Se você abrir o terminal do vscode e notar que o prompt for algo assim `root@f1234567:/app#` em vez de `(seu-usuario@seu-usuario)`. Significa que deu certo.
* Agora a IDE opera de dentro do container com todas as ferramentas instaladas.

## Build Final Dockerfile para produção
Para diminuir o tamanho da imagem ao buildar a aplicação, será seguida passo a passo do Dockerfile Multi-Stage seguindo a lógica: "Você desenvolve no container pesado (JDK), mas gera uma imagem final leve (JRE) que conterá apenas o necessário para rodar a aplicação."

### Crie um arquivo chamado `Dockerfile` na raiz do seu projeto e cole o seguinte script: <br>

```
# 1 Etapa: Compilação
# Será utilizado o JDK (Java Development Kit) para transformar o código em .jar
FROM eclipse-temurin:21-jdk-alpine AS builder

# Define o diretório de trabalho
WORKDIR /build

# Copia os arquivos de configuração do Maven/Gradle primeiro para otimizar cache
COPY .mvn/ .mvn
COPY mvnw pom.xml ./

# Atribui permissão de execução ao arquivo
RUN chmod +x mvnw

RUN ./mvnw dependency:go-offline

# Copia o código fonte e gera o executável
COPY src ./src
RUN ./mvnw clean package -DskipTests

# 2 Etapa: Execução
# Será utilizado o JRE (Java Runtime Environment) para reduzir o tamanho final, já que o JRE é muito menor
FROM eclipse-temurin:21-jre-alpine

# Define o diretório de trabalho para execução
WORKDIR /app

# Copia o artefato do estágio anterior
COPY --from=builder /build/target/*.jar app.jar

# Define a porta que será executada
EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Buildar e Rodar
Navege até a raiz do seu projeto onde está o Dockerfile e execute este comando: <br>
`docker build -t <nome_imagem_build> .`

* `build`: Cria uma nova imagem a partir de um arquivo (Dockerfile).
* `-t <nome_imagem_build>`: Atribui um nome personalizado (tag) para a imagem criada a fim de não gerenciar ela pelo o ID.
* `.`: Indica que o contexto do build (arquivos src, pom.xml, outros) está na pasta atual.

Após o build terminar, você poderá subir o container usando o comando: <br>
* Rodar localmente via terminal: `docker run -it --rm --name <nome_container> <nome_imagem_build>`
* Rodar via Servidor (Web): `docker run -p 8080:8080 --rm --name <nome_container> <nome_imagem_build>`

* `run`: Cria e inicia um container a partir de uma imagem.
* `-it`: Cria um terminal interativo.
* `--rm`: Deleta o container assim que encerrado.
* `--name <nome_container>`: Atribui um nome específico ao container rodando. Ex: Nome do container => `app-terminal`
* `<nome_imagem_build>`: Indica a imagem que deve ser utilizada para criar o container.
* `-p 8080:8080`: Mapeia a porta 8080 da sua máquina para a porta 8080 do container.