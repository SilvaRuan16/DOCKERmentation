# DOCKER PARA BACKEND
Este documento tem como objetivo explicar e fornecer modelos de container para ambientes de desenvolvimento com docker, evitando ouvir a famosa frase: "Na minha máquina funciona".

Será fornecido algumas ferramentas ao longo desta ferramenta, sendo elas:
| Ferramenta    | Versão    |
| :--           | :--       |
| `Java`        | 25        |
| `PHP`         | 8.2       |
| `Node`        | latest    |
| `Postgres`    | latest    |
| `Mysql`       | latest    |
| `MariaDB`     | latest    |
| `MongoDB`     | latest    |
| `ScyllaDB`    | latest    |

---
## COMANDOS GLOBAIS DE MONITORAMENTO
| Ação            | Comando                               | Descrição                                              |
| :---            | :---                                  | :---                                                   |
| Status Global   | `docker ps -a`                        | Lista todos os containers e seus estados (Up, Exited)  |
| Uso de Recursos | `docker stats`                        | Monitora consumo de CPU e Memória em tempo real        |
| Ver Imagens     | `docker images`                       | Lista todas as imagens baixadas localmente             |
| Ver Logs        | `docker logs -f <nome_do_container>`  | Mostra todos os logs do container em execução          |
---
## COMANDOS DE CONTROLE
| AÇÃO            | COMANDO                               | DESCRIÇÃO                                              |
| :---            | :---                                  | :---                                                   |
| Parar           | `docker stop <id_ou_nome>`            | Para o container salvando oque precisa                 |
| Iniciar         | `docker start <id_ou_nome>`           | Inicia um container que foi parado                     |
| Reiniciar       | `docker restart <id_ou_nome>`         | Reinicia o container                                   |
| Matar           | `docker kill <id_ou_nome>`            | Mata a execução do container                           |
| Reinicia Docker | `sudo systemctl restart docker`       | Reinicia o Docker da máquina                           |  
---
## COMANDOS DE LIMPEZA
| Ação            | Comando                               | Descrição                                              |
| :---            | :---                                  | :---                                                   |
| Remover         | `docker rm <id_ou_nome>`              | Remove um container parado                             |
| Deletar         | `docker rmi <id_ou_nome_da_imagem>`   | Deleta uma imagem da sua máquina                       |
| Limpeza Geral   | `docker system prune`                 | Apaga todos os containers, redes e cache não utilizados|
---
## COMANDOS DE ACESSO AO BASH
| Ação             | Comando                                    | Descrição                                         |
| :---             | :---                                       | :---                                              |
| Acessar Bash     | `docker exec -it <nome_do_container> bash` | Abre terminal interativo bash do container        |
| Acessar SH       | `docker exec -it <nome_do_container> sh`   | Abre um terminal simples                          |
| Sair do Terminal | `exit`                                     | Sai do modo interativo e volta para sua máquina   |

## GERENCIAMENTO DO DOCKER - COMPOSE
| Comando                        | Ação                                                      |
| :---                           | :---                                                      |
| `docker compose up --build -d` | Cria e inicia os contêiners                               |
| `docker compose build`         | Realiza a etapa de build das imagens que serão utilizadas |
| `docker compose logs -f app`   | Visualiza os logs dos contêiners                          |
| `docker compose restart`       | Reinicia os contêiners                                    |
| `docker compose ps`            | Lista os contêiners                                       |
| `docker compose scale`         | Permite aumentar o número de réplicas de um contêiner     |
| `docker compose start`         | Inicia os contêiners                                      |
| `docker compose stop`          | Paralisa os contêiners                                    |
| `docker compose down -v`       | Paralisa e remove todos os contêiners (tudo)              |