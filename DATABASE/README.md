# Database
Este tópico de documentação tem como função fornecer como utilizar banco de dados via docker a fim de deixar a máquina limpa de ferramentas e configurações de ambientes. Como exemplo, será fornecido comandos para a utilização de Postgresql, mysql e mariadb.

## Postgresql
É um sistema de gerenciamento de banco de dados relacional objeto (SGBD), tendo amplo espaço no mercado pela sua robustez, confiabilidade, desempenho avançado e conformidade com os padrões SQL.

### Comando para utilizar o Postgresql via container

```
docker run --name postgres-db \
-e POSTGRES_PASSWORD=minhasenha \
-e POSTGRES_USER=meuusuario \
-e POSTGRES_DB=meubanco \
-p 5432:5432 \
-d postgres:latest
```

#### Explicação de cada comando
* `docker run`: Executa um novo container a partir de uma imagem.
* `--name postgres-db`: Define o nome do container.
* `-e POSTGRES_PASSWORD=minhasenha`: Define a senha do usuário.
* `-e POSTGRES_USER=meuusuario`: Define usuário padrão do banco.
* `-e POSTGRES_DB=meubanco`: Define o nome do banco de dados criado automaticamente.
* `-p 5432:5432`: Mapeia a porta do host para a porta do container.
* `-d postgres:latest`: Define o container em segundo placo e a imagem que será usada mais sua versão.

## Mysql
É um sistema de gerenciamento de banco de dados relacional (SGBDR), sendo muito popular por conta da sua compatibilidade, desempenho e escalabilidade.

### Comando para utilizar o Mysql via container

```
docker run --name mysql-db \
-e MYSQL_ROOT_PASSWORD=minhasenha \
-e MYSQL_USER=seuusuario \
-e MYSQL_DATABASE=meubanco \
-p 3333:3306 \
-d mysql:latest
```

#### Explicação de cada comando
* `docker run`: Executa um novo container a partir de uma imagem.
* `--name mysql-db`: Define o nome do container.
* `-e MYSQL_ROOT_PASSWORD=minhasenha`: Define a senha do usuário.
* `-e MYSQL_USER=meuusuario`: Define usuário padrão do banco.
* `-e MYSQL_DATABASE=meubanco`: Define o nome do banco de dados criado automaticamente.
* `-p 3305:3306`: Mapeia a porta do host para a porta do container.
* `-d mysql:latest`: Define o container em segundo placo e a imagem que será usada mais sua versão.

## Mariadb


### Comandos para utilizar o Mariadb via container
```
docker run --name maria-db \
-e MARIADB_ROOT_PASSWORD=suasenha \
-e MARIADB_DATABASE=meubanco \
-e MARIADB_USER=seuusuario \
-e MARIADB_PASSWORD=suasenha \
-p 3306:3306 \
-d mariadb:latest
```

### Rodar database via bash
Você executar e manipular a database via terminal bash do container, para isso será necessário rodar o seguinte comando: `sudo docker exec -it <nome_container> bash`.

Explicação: Você roda um terminal bash iterativo do container informado.

### Descobrir ip do container para conectar ao DBeaver
Para descobrir o ip do seu container para configurar ambiente DBeaver, você poderá executar o seguinte comando: `sudo docker inspect <nome_container> | grep IPAddress`.

### Como acessar o banco no DBeaver
Para ter acesso ao seu banco de dados em um gerenciador de banco de dados, você precisará seguir alguns passos:
* Ter o DBeaver instalado.
* Quando abrir ele, adicione uma nova conexão e escolha sua database.
* Na interface de conexão, você deverá inserir todas as informações que você usou para criar o container, incluindo o ip capturado no container.
* Acesse uma área chamada Driver properties e altere uma linha chamada `allowPublicKeyRetrieval` de false para true.