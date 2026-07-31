[← README](../README.md)

# Hosts Docker

Hosts dedicados a rodar aplicações diversas em containers Docker. Cada host é
**independente** (sem orquestração/cluster entre eles, ex.: Swarm) e gerencia
suas próprias aplicações via Docker Compose.

Este documento lista as aplicações em execução em cada host. Aplicações que
ganharem documentação própria são referenciadas aqui com link cruzado.

## Hosts

| Hostname    | IP           |
| ----------- | ------------ |
| `docker-01` | 172.16.0.165 |
| `docker-02` | 172.16.0.166 |
| `docker-03` | 172.16.0.167 |

## docker-01

### Aplicações em execução

| Aplicação     | Diretório base                    | Compose | Containers | Descrição                                                                                                                                                       |
| ------------- | --------------------------------- | ------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **waha-plus** | `/root/docker-services/waha-plus` | Sim     | 3          | Plataforma de conexão para WhatsApp (pesquisar "WAHA WhatsApp HTTP API" para mais detalhes). Usado para testes de aplicação com WhatsApp. Ver [Waha](./waha.md) |
| **n8n**       | `/root/docker-services/n8n`       | Sim     | 1          | Aplicação de automação, solicitada pelo Leo em junho/2026                                                                                                       |
| **rundeck**   | `/root/docker-services/rundeck`   | Sim     | 2          | Servidor de CRON — agendamento e execução de tarefas. Ver [Rundeck](./rundeck.md)                                                                               |

### Projetos desativados

Presentes em `/root/docker-services/`, mas sem execução no momento:

| Projeto   | Descrição                                                                                                                                                                            | Em execução | Pode excluir |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------- | ------------ |
| `lab_git` | Testes de deploy automático via GitHub Actions. Recurso já implantado nos apps de produção e homologação                                                                             | Não         | Sim          |
| `waha`    | Versão _community_ do Waha, instalada na primeira fase dos testes. Substituída pelo `waha-plus` (também neste host) e, posteriormente, pela versão final em produção (em outro host) | Não         | Sim          |

## docker-02

### Aplicações em execução

| Aplicação     | Diretório base                    | Compose | Containers | Descrição                                                                                                    |
| ------------- | --------------------------------- | ------- | ---------- | -------------------------------------------------------------------------------------------------------------- |
| **waha-plus** | `/root/docker-services/waha-plus` | Sim     | 3          | Plataforma de conexão para WhatsApp — **ambiente de produção**. Ver [Waha](./waha.md) |

## docker-03

### Aplicações em execução

| Aplicação             | Diretório base                    | Compose | Containers | Descrição                                                                                                        |
| --------------------- | --------------------------------- | ------- | ---------- | -------------------------------------------------------------------------------------------------------------------- |
| **waha-plus (worker)** | `/root/docker-services/waha-plus` | Sim     | 1          | Worker do Waha — **ambiente de produção**. Ver [Waha](./waha.md) |
