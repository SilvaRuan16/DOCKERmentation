# Multiplataforma (Dart e Flutter)
Este guia documenta o processo de utilização de um container docker para isolar o ambiente de desenvolvimento, permitindo criar, testar e buildar sem precisar instalar as ferramentas na máquina física.

## 1. Configuração do Ambiente CLI (Aliases)
Será criada uma alias para criar um apelido ao comando e facilitar o uso no terminal (`.bashrc` ou `zshrc`).
Obs: Será utilizada a imagem (NÃO OFICIAL) cirrusci/flutter:stable por ser a mais estável e mantida para ambientes de CI/CD e Docker.
```
# Função padrão para o dia a dia
d-flutter() {
    docker run -it --rm -v "$(pwd)":/app -w /app cirrusci/flutter:stable flutter "$@" && sudo chown -R $USER:$USER .
}

# Função para atualizar a imagem oficial e limpar tudo (O "Reset de Fábrica")
# Use isso sempre que aparecerem erros bizarros de Matrix4 ou SDK
d-flutter-update() {
    echo "Atualizando imagem Docker do Flutter Stable..."
    docker pull cirrusci/flutter:stable
    echo "Limpando caches locais do projeto..."
    rm -rf .dart_tool/ build/ pubspec.lock
    docker run -it --rm -v "$(pwd)":/app -w /app cirrusci/flutter:stable flutter doctor
}

d-shell() {
    docker run -it --rm -v "$(pwd)":/app -w /app cirrusci/flutter:stable bash && sudo chown -R $USER:$USER .
}

```
Explicação do principais parâmetros:
* `-u $(id -u):$(id -g)`: Faz com que os arquivos criados pelo Docker pertença ao seu usuário da máquina e não ao root do container.
* `-v "$(pwd)":/app`: Faz o mapeamento da pasta atual da sua máquina para o diretório /app do container.
* `--rm`: Garante que o container seja deletado ao finalizar o comando, economizando espaço em disco.

## 2. Comandos de Gerenciamento e Uso
| Ação | Comando |
| :--- | :--- |
| Criar novo app | `d-flutter create <nome_do_app>` |
| Baixar dependências | `d-flutter pub get` |
| Limpar build/cache | `d-flutter clean` |
| Verificar saúde do SDK | `d-flutter doctor` |
| Licenças Android | `d-flutter doctor --android-licenses` |
| Analisar projeto em busca de erro | `d-flutter analyze` |
| Corrigir permissões (dentro do terminal do container) | `sudo chown -R $USER:$USER .` |
| Abrir terminal do container | `d-shell` |

Obs: Caso você rodar o `d-flutter doctor` e ver vários problemas [X], não precisa se preocupar, pois só será necessário em tarefas extremamente grandes. <br>
Obs 2: Quando manipular o projeto dentro do container, você deverá executar `sudo chown -R $USER:$USER .` para enviar as mudanças para a máquina local.

## 3. Desenvolvendo com VS Code (Extensão: Dev Containers)
É recomendado instalar a extensão `Dev Containers` para ter acesso a suporte da IDE (Autocomplete, Refatoração e Linting).

### Configuração do Projeto (Via terminal do container)
* #### 1. Crie uma pasta chamada `.devcontainer` na raiz do seu projeto usando: `mkdir .devcontainer && cd ./.devcontainer`.
* #### 2. Crie o arquivo `devcontainer.json` dentro do diretório .devcontainer usando: `touch devcontainer.json` e depois `nano devcontainer.json` ou somente o `nano devcontainer.json`.
* #### 3. Note que o seu usuário externo não tem permissão para escrever arquivos, então no terminal dentro do container, no usuário root, execute estes comandos para conseguir escrever e modificar arquivos (Manutenção rápida via terminal do container. `apt update && apt install nano -y`). Obs: Como o container usa a flag --rm, qualquer ferramenta instalada via apt será removida assim que fechar o terminal (exit).
* #### 3. Coloque este código abaixo dentro do arquivo `devcontainer.json`:
```
{
  "name": "Flutter Dev Container",
  "image": "cirrusci/flutter:stable",
  "remoteUser": "root",
  "workspaceFolder": "/app",
  "mounts": [
    "source=${localWorkspaceFolder},target=/app,type=bind"
  ],
  "customizations": {
    "vscode": {
      "extensions": [
        "dart-code.flutter",
        "dart-code.dart-code"
      ],
      "settings": {
        "dart.flutterSdkPath": "/sdks/flutter"
      }
    }
  },
  "postCreateCommand": "git config --global --add safe.directory /sdks/flutter && chown -R 1000:1000 /app"
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

## 4. Rodando o App (Flutter WEB) pelo terminal do container
"Pré-requisito: Limpeza de Cache" Antes de iniciar a aplicação, é necessário realizar uma limpeza dos arquivos temporários. Como o processo de criação inicial pode gerar um cache inconsistente, este procedimento garante que a execução ocorra sem conflitos de permissão ou erros de compilação.
* Limpe o cache do Pub (Confirme com y se ele fizer alguma pergunta): <br>
`flutter pub cache clean`
* Remova os arquivos de Build e Lock:
`rm -rf build/ .dart_tool/ pubspec.lock`
* Force o download limpo das dependências:
`flutter pub get`
* Após isso, siga as instruções abaixo:
Por o Docker criar container de forma isolada, a visualização precisará ser feita via Web Server. <br>
`flutter run -d web-server --web-hostname 0.0.0.0 --web-port 8080`. <br>
Obs: Após executar o comando anterior, vai aparecer esta mensagem `Your application running on port 8080 is available. See all forwarded ports`, será necessário clicar no botão azul `Open in Browser` ou acesse em seu navegador: `http://localhost:8080` | `http://localhost:8080/#/`.

## 5. Docker para para Produção (Dockerfile)
Este Dockerfile foi adaptado para gerar builds leves utilizando Multi-stage Build. Obs: Estes comentários são para explicar o passo a passo, mas pode ficar a vontade para remove-los. Todo comentário começa com este símbolo (`#`)

```
############################
#  STAGE-1: BUILD PROCESS  #
############################


# Define a imagem base
# Flutter e Dart instalados
# Atribui apelido 'build' para captura de dados
FROM cirrusci/flutter:stable AS build

# Cria e entra na pasta /app dentro do container
# Todas as atividades acontecerão neste diretório
WORKDIR /app

# Copia todos os arquivos onde está o Dockerfile
# Move para dentro da pasta /app do container
COPY . .

# Baixa as dependências do projeto
# Compila o app para a plataforma web
# Resultado gerado em build/web
RUN flutter pub get && flutter build web

#############################
#  STAGE-2: RUNNER PROCESS  #
#############################

# Inicia uma imagem Nginx
# Servidor web
# Minimalista
FROM nginx:alpine

# Vai até o estágio anterior (--from=build)
# Pega apenas a pasta onde o site foi compilado
# Copia para a pasta padrão do Nginx
# Deixa o SDK do Flutter e o código-fonte para trás
COPY --from=build /app/build/web /usr/share/nginx/html

# Informa a porta de conexão do container
EXPOSE 80

# Define o comando que mantém o servidor rodando
# daemon off para que o container continue ativo
CMD ["nginx","-g","daemon off;"]
```

## 6. Ciclo de Vida e Manutenção
| Ação | Comando |
| :--- | :--- |
| Listar ativos | `docker ps` |
| Ver todos | `docker ps -a` |
| Parar | `docker stop <ID_ou_Nome>` |
| Iniciar | `docker start <ID_ou_Nome>` |
| Reiniciar | `docker restart <ID_ou_Nome>` |
| Logs | `docker logs -f <ID_ou_Nome>` |

## 7. Processo para Buildar a Aplicação (Web e Android)
Só será possível buildar a aplicação de duas formas, sendo web e android, pois elas não precisam necessariamente de todas as configurações e dependências do flutter. 
Enquanto buildar para window, linux e ios precisa de todas as ferramentas necessárias (nesses casos, recomendo baixar as ferramentas diretamente na sua máquina).

### Buildar construção Dockerfile e executa
| Ação | Comando |
| :--- | :--- |
| Buildar | `docker build -t meu-app-flutter:prod .` |
| Rodar | `docker run -d -p 9000:80 --name app-producao meu-app-flutter:prod` |
| Host | `http://localhost:9000` |
| Baixar imagem manualmente | `docker pull nginx:alpine` |
| Reiniciar docker | `sudo systemctl restart docker` |

### Gerar o app
| Ação | Comando |
| :--- | :--- |
| Android | `d-flutter build apk` |
| Web | `d-flutter build web` |