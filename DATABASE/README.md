# Database
Está sessão fornecerá comando de execução para banco de dados SQL e NOSQL em ambientes dockerizados, a fim de fornecer e manter a máquina limpa de configurações diretamente na máquina.

Será abordado banco de dados:
* SQL: Postgresql, Mysql/MariaDB
* NOSQL: MongoDB e ScyllaDB

## Postgresql
É um sistema de gerenciamento de banco de dados relacional objeto (SGBD), tendo amplo espaço no mercado pela sua robustez, confiabilidade, desempenho avançado e conformidade com os padrões SQL.

### Comando para utilizar o Postgresql via container

```bash
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

```bash
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
O MariaDB é um sistema de gerenciamento de banco de dados relacional de código aberto, desenvolvido pelos criadores originais do MySQL como uma alternativa robusta, rápida e totalmente compatível com ele. Ele é amplamente utilizado no mercado devido ao seu alto desempenho, recursos avançados de segurança e uma comunidade ativa que garante constantes inovações.

### Comandos para utilizar o Mariadb via container
```bash
docker run --name maria-db \
-e MARIADB_ROOT_PASSWORD=suasenha \
-e MARIADB_DATABASE=meubanco \
-e MARIADB_USER=seuusuario \
-e MARIADB_PASSWORD=suasenha \
-p 3306:3306 \
-d mariadb:latest
```

### Explicação de cada comando

* `docker run`: Executa um novo container a partir de uma imagem.
* `--name maria-db`: Define o nome do container.
* `-e MARIADB_ROOT_PASSWORD=suasenha`: Define a senha de administrador (root) do banco de dados.
* `-e MARIADB_DATABASE=meubanco`: Define o nome do banco de dados criado automaticamente.
*` -e MARIADB_USER=seuusuario`: Define o usuário padrão adicional criado no banco.
* `-e MARIADB_PASSWORD=suasenha`: Define a senha para o usuário padrão adicional criado.
* `-p 3306:3306`: Mapeia a porta do host para a porta do container.
* `-d mariadb:latest`: Define o container em segundo plano e a imagem que será usada mais sua versão.

# Nosql

## MongoDB (Documentar)
O MongoDB é um banco de dados NoSQL orientado a documentos que armazena informações em formatos flexíveis semelhantes ao JSON, em vez de tabelas e linhas tradicionais. Ele é altamente escalável e ideal para lidar com grandes volumes de dados não estruturados, permitindo modificações rápidas na estrutura dos dados sem a necessidade de migrações complexas.

```bash
docker run -d \
--name mongo-db \ 
-p 27017:27017 \ 
-v mongo_data:/data/db \
mongo:latest
```

### Explicação de cada comando

* `docker run`: Executa um novo container a partir de uma imagem.
* `-d`: Define o container em segundo plano (detached mode).
* `--name mongo-db`: Define o nome do container.
* `-p 27017:27017`: Mapeia a porta do host para a porta do container.
* `-v mongo_data:/data/db`: Cria e mapeia um volume nomeado para persistir os dados do banco fora do ciclo de vida do container.
* `mongo:latest`: Define a imagem que será usada mais sua versão.

## ScyllaDB (Colunar)
O ScyllaDB é um banco de dados NoSQL de colunas largas e distribuído, projetado para oferecer ultra-baixa latência e altíssimo rendimento. Ele foi desenvolvido em C++ como uma alternativa drop-in de alto desempenho para o Apache Cassandra, aproveitando ao máximo a arquitetura de hardware moderna.

### Comando para utilizar o ScyllaDB via container

```bash
docker run --name scylla-db \
-p 9042:9042 \
-d scylladb/scylla:latest \
--smp 1 \
--memory 1G
```

### Explicação de cada comando

* `docker run`: Executa um novo container a partir de uma imagem.
* `--name scylla-db`: Define o nome do container.
* `-p 9042:9042`: Mapeia a porta nativa de transporte do CQL (Cassandra Query Language) do host para o container.
* `-d scylladb/scylla:latest`: Define o container em segundo plano e a imagem oficial do ScyllaDB que será usada mais sua versão.
* `--smp 1`: Restringe o ScyllaDB a utilizar apenas 1 thread de CPU (ideal para ambientes de desenvolvimento local).
* `--memory 1G`: Limpa e limita a quantidade de memória RAM disponível para o container (evitando que ele consuma todos os recursos da máquina).