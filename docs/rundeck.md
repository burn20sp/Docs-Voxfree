[← README](../README.md)

# Rundeck

## Visão geral

| Campo | Valor |
|---|---|
| **Nome** | Rundeck |
| **Descrição** | Servidor de CRON — agendamento e execução de tarefas ([rundeck.com](https://www.rundeck.com/)) |
| **IP** | 172.16.0.165 |
| **Acesso** | <http://172.16.0.165:4440> |
| **Plataforma** | Docker, no servidor [`docker-01`](./docker.md) (172.16.0.165) |

## Credenciais de acesso

 - **Usuário:** admin
 - **Senha:** admin

## Infraestrutura / arquivos

| Item | Local |
|---|---|
| Diretório base (docker) | `docker-01:/root/docker-services/rundeck/` |
| Dados persistentes da aplicação | `./data` (dentro do diretório base) |

## Comunicação com hosts (SSH)

O Rundeck se comunica com os hosts gerenciados via SSH usando chave. Para um
host ser gerenciável, a chave pública abaixo precisa estar autorizada nele
(`authorized_keys` do usuário usado pelo Rundeck):

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFucvA10XxcOw96WMnBFk5NitlMFlzr9pEtmO+uWG6R8 command@rundeck
```

## Organização: Projects

Primeiro nível de organização do Rundeck — cada Project agrupa hosts e
Tarefas relacionadas. Atualmente existem dois:

### `asterisk`

Hosts: discadores, receptivos, sip-proxy.

### `Services` (mais recente)

Hosts: `http-services` — trata dos scripts auxiliares do APP executados em
segundo plano, que antes rodavam no `web28`.

## Tarefas (Jobs)

- São as execuções agendadas, organizadas em grupos.
- Cada Tarefa pode executar um ou mais comandos e/ou scripts em um
  determinado grupo de hosts.

## Referências

- Documentação oficial: <https://docs.rundeck.com/>
- [Hosts Docker](./docker.md) — demais aplicações rodando em `docker-01`
