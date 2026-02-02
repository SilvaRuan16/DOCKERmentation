# Ferramentas Dockerizadas para Backend
Esta parte do guia mostrará como criar ambientes dockerizados para desenvolvimento, build e execução de suas aplicações, sendo as ferramentas mencionadas: Java e Python.

## Java
Java é uma linguagem de programação orientada a objetos fortemente tipada, suas principais caracteristicas são sua segurança e robustez, desempenho, independente de plataforma e como mencionada anteriormente, orientação a objetos.

## 1. Configuração do ambiente CLI
Será criada uma função para facilitar a execução dos programas no terminal (`.bashrc` ou `zshrc`).
```
# Função para utilizar no dia a dia
d-java() {
  docker run -it --rm -v "$(pwd)":/app -v "$HOME/.m2":/root/.m2 -w /app eclipse-temurin:21-jdk bash
}
```
### Explicação dos trechos do comando docker:
* `-it`: Mantém o terminal interativo para utilizar o bash.
* `--rm`: Remove o container de forma automática assim que o usuário sair dele, mantendo o seu sistema limpo.
* `-v "$(pwd)":/app`: Monta sua pasta atual no diretório `/app` do contaiener.
* `-v "$HOME/.m2":/root/.m2`: Compartilha o cache do maven para não precisar baixar todas as dependências do zero todas as vezes.
* `-w /app`: Define o diretório de trabalho inicial como `/app`.
* `eclipse-temurin:21-jdk`: Utiliza uma imagem oficial do OpenJDK 21 pela distribuição Temurin.

Após isso, você poderá executar esse comando toda vez que for escrito `d-java`.

## Ciclo de vida Básico
Caso você queira um melhor gerenciamento sobre o seu container, será necessário remover a flag `--rm` e inserir uma nova flag `--name` para ganhar mais controle sobre o container.

| Comandos | Ação |
| :--- | :--- |
| `docker ps` | Lista os containers ativos no momento. |
| `docker ps -a` | Lista todos os containers e seus estados (Up, Exited). |
| `docker stop <nome_ou_id>` | Para o container enviando um sinal de desligamento. |
| `docker start <nome_ou_id>` | Reinicia um container que foi parado. |
| `docker restart <nome_ou_id>` | Para e inicia novamente (útil se o Java travar). |
| `docker rm <nome_ou_id>` | Remove o container permanentemente do disco. |

## Comandos de Inspeção e Uso
Possui casos onde mesmo o container sendo executado em segundo plano usando a flag `-d`, será preciso interagir com o mesmo.

### Executar comandos em um container que já está rodando
Caso você queira abrir um terminal no container do Java, execute este comando: <br>
`docker exec -it <nome_do_container> bash`

### Ver os logs
Recomendado para debugar aplicações Java que estão rodando por trás dos panos: <br>
`docker logs -f <nome_do_container>` <br>
Obs: A flag -f (follow) faz com que o terminal fique seguindo o log em tempo real.

## Dockerfile para aplicações em Java e Desenvolvendo com VS Code (Extensão: Dev Containers)
No conteúdo mostrado logo acima, foi fornecido o ambiente de desenvolvimento Java sem ter essas ferramentas necessáriamente baixadas em sua máquina física. Agora será mostrado o passo a passo de como Buildar e Rodar uma aplicação Java atráves do Dockerfile.

É recomendado instalar a extensão `Dev Containers` para ter acesso a suporte da IDE (Autocomplete, Refatoração e Linting).

### Configuração do Projeto (Via terminal do container)
* #### 1. Crie uma pasta chamada `.devcontainer` na raiz do seu projeto usando: `mkdir .devcontainer && cd ./.devcontainer`.
* #### 2. Crie o arquivo `devcontainer.json` dentro do diretório .devcontainer usando: `touch devcontainer.json` e depois `nano devcontainer.json` ou somente o `nano devcontainer.json`.
* #### 3. Note que o seu usuário externo não tem permissão para escrever arquivos, então no terminal dentro do container, no usuário root, execute estes comandos para conseguir escrever e modificar arquivos (Manutenção rápida via terminal do container. `apt update && apt install nano -y`). Obs: Como o container usa a flag --rm, qualquer ferramenta instalada via apt será removida assim que fechar o terminal (exit).
* #### 3. Coloque este código abaixo dentro do arquivo `devcontainer.json`:
```
{
  "name": "Java 21 Dev Container",
  "image": "mcr.microsoft.com/devcontainers/java:21-eclipse-temurin",
  "remoteUser": "vscode",
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
  },
  "postCreateCommand": "sudo chown -R vscode:vscode /workspaces"
}
```
### Como utilizar:
* Abra o projeto no Vs Code ou no terminal fora do container, vá para o caminho do projeto espelhado e use o comando `code .` para abrir o vscode.
* Agora, você tem 3 opções de abrir o container do projeto no vscode:
  * Forma A (Quadrado azul): No canto inferior esquerdo do seu Vs code, existe um ícone azul (parece um ><). Clique nele e selecione "Reopen in Container".
  * Forma B (O atalho): Aperte Ctrl + Shift + P e digite: `Dev Containers: Reopen in Container`.
  * Forma C (A notificação): Se o arquivo .devcontainer estiver no projeto, o Vscode costuma mostrar um balão no canto inferior direito dizendo: "Folder contains a Dev Container configuration file. Reopen to develop in a container?". Clique em `Reopen`.
* Como saber se deu certo? Se você abrir o terminal do vscode e notar que o prompt for algo assim `root@f1234567:/app#` em vez de `(seu-usuario@seu-usuario)`. Significa que deu certo.
* Vs Code agora opera de dentro do container com todas as ferramentas instaladas.

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
`docker build -t meu-app-java .`

* `-t meu-app-java`: Atribui um nome (tag) para a imagem criada para que não precise gerenciar ela pelo o ID.
* `.`: Indica que o contexto do build (arquivos src, pom.xml, outros) está na pasta atual.

Após o build terminar, você poderá subir o container usando o comando: <br>
`docker run -d -p 8080:8080 --name app-execucao meu-app-java`

* `-d`: (Detached), roda o container em segundo plano para deixar o terminal livre para uso.
* `-p 8080:8080`: Mapeia a porta 8080 da sua máquina física para a porta 8080 do container.
* `--name app-execucao`: Atribui um nome ao container para facilitar o gerenciamento.