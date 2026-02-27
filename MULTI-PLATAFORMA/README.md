# Multiplataforma (Flutter)
Este guia documenta o processo de utilização de um container docker para isolar o ambiente de desenvolvimento, permitindo criar, testar e buildar sem precisar instalar as ferramentas na máquina física.

## 1. Criando Pacote Pai da Aplicação
Para que seja possível criar um projeto em Flutter sem depender de ferramentas instaladas na máquina física, vamos precisar mudar a nossa abordagem de criação do projeto. Geralmente é criado o pacote da aplicação e logo após é realizada uma configuração do ambiente de desenvolvimento docker. Mas neste caso vamos precisar inverter essa lógica, primeiro vamos configurar o pacote pai e depois criar o projeto.

* 1. Crie uma pasta pai onde será inserida o projeto.
* 2. Dentro da pasta pai, será necessário criar uma nova pasta chamada `.devcontainer`.
* 3. Dentro da pasta .devcontainer, será necessário criar um arquivo chamado `devcontainer.json`.
* 4. Cole o seguinte conteúdo no arquivo devcontainer.json:

```
{
  "name": "Flutter Dev Container",
  "image": "cirrusci/flutter:stable",
  "remoteUser": "root",
  "workspaceFolder": "/app",
  "customizations": {
    "vscode": {
      "extensions": [
        "Dart-Code.flutter",
        "Dart-Code.dart-code"
      ],
      "settings": {
        "dart.flutterSdkPath": "/sdks/flutter",
        "dart.sdkPath": "/sdks/flutter/bin/cache/dart-sdk"
      }
    }
  },
  "postCreateCommand": "git config --global --add safe.directory /sdks/flutter && flutter doctor"
}
```

## 2. Abrir IDE
Quando realizado a 1 etapa, agora vamos escolher uma IDE conforme sua preferência, no caso desta documentação será escolhido o VScode para Multiplataforma pela quantidade de extensões para auxiliar no desenvolvimento, porém fiquem a vontade para escolher outras IDEs como Intelij, Android Studio, etc.

Obs: Será necessário extensões de desenvolvimento com Docker, como Dev Container e outros.

Se o arquivo .devcontainer estiver na pasta pai, a IDE costuma mostrar um balão no canto inferior direito dizendo: "Folder contains a Dev Container configuration file. Reopen to develop in a container?". Clique em `Reopen`.

Como saber se deu certo? Se você abrir o terminal e notar que o prompt for algo assim `root@f1234567:/app#` em vez de `(seu-usuario@seu-usuario)`. Significa que deu certo.

## 3. Criar Projeto Flutter
Agora que o ambiente está instalado e configurado, vamos criar o projeto flutter. Aperte Ctrl + Shift + P e escreva `Flutter: New Project`. Após isso será necessário configurar e logo em seguida estará pronto.

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

## Comandos de Gerenciamento e Uso do Flutter
| Ação                              | Comando                             |
| :---                              | :---                                |
| Baixar dependências               | `flutter pub get`                   |
| Limpar build/cache                | `flutter clean`                     |
| Verificar saúde do SDK            | `flutter doctor`                    |
| Licenças Android                  | `flutter doctor --android-licenses` |
| Analisar projeto em busca de erro | `flutter analyze`                   |

## Gerar o app (Terminal do container)
| Ação          | Comando                       |
| :---          | :---                          |
| Android       | `flutter build apk --release` |
| Web           | `flutter build web`           |
| Desktop Linux | `flutter build linux`         |
