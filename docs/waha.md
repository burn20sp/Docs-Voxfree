[← README](../README.md)

# Waha (waha-plus)

Plataforma de conexão para WhatsApp (pesquisar "WAHA WhatsApp HTTP API" para
mais detalhes sobre o produto). Roda em 3 hosts, com propósitos distintos:
testes (`docker-01`) e produção (`docker-02` + `docker-03`).

## Visão geral por host

| Host | IP | Ambiente | Papel | Containers |
|---|---|---|---|---|
| [`docker-01`](./docker.md) | 172.16.0.165 | Testes | Instância completa | 3 |
| [`docker-02`](./docker.md) | 172.16.0.166 | **Produção** | Instância completa (+ Postgres/Redis) | 3 |
| [`docker-03`](./docker.md) | 172.16.0.167 | **Produção** | Worker adicional | 1 |

Em todos os hosts:

| Campo | Valor |
|---|---|
| **Diretório base** | `/root/docker-services/waha-plus` |
| **Gerenciamento** | Docker Compose |

## Acesso

| Host | Porta | Dashboard | API/Swagger |
|---|---|---|---|
| `docker-01` (testes) | 3000 | `/dashboard` | `/` |
| `docker-02` (produção) | 3000 | `/dashboard` | `/` |
| `docker-03` (worker) | — | Desabilitado (`WAHA_DASHBOARD_ENABLED=False`) | Desabilitado (`WHATSAPP_SWAGGER_ENABLED=False`) |

Exemplos:
- Testes — dashboard: `http://172.16.0.165:3000/dashboard` / API: `http://172.16.0.165:3000/`
- Produção — dashboard: `http://172.16.0.166:3000/dashboard` / API: `http://172.16.0.166:3000/`

## Credenciais

> **Atenção**: por padrão não é recomendado documentar credenciais em texto
> plano. Estão registradas aqui a pedido, para referência operacional —
> restrinja o acesso a este arquivo/repositório e trate como informação
> sensível.

### docker-01 (testes)

| Variável | Valor |
|---|---|
| `WAHA_API_KEY` | `bce168adbd94e05f3c5ef43f813202bb197e243892b38e1e6bb9c42f2e3dba8e` |
| `WAHA_API_KEY_PLAIN` | `HpSLMn77NYwsATcNrWZptvoGNJnsrfQD6pC4UDs2beSEeZAEaACaAPYyyo5tRmdt` |
| `WAHA_DASHBOARD_USERNAME` | `admin` |
| `WAHA_DASHBOARD_PASSWORD` | `86b20c3028014f5581f0e4c624ef1888` |
| `WHATSAPP_SWAGGER_USERNAME` | `admin` |
| `WHATSAPP_SWAGGER_PASSWORD` | `86b20c3028014f5581f0e4c624ef1888` |
| Postgres | `postgres://waha:e4c3d1a4cb2941327f380450456@postgres:5432/waha?sslmode=disable` |

### docker-02 (produção)

| Variável | Valor |
|---|---|
| `WAHA_API_KEY` | `bce168adbd94e05f3c5ef43f813202bb197e243892b38e1e6bb9c42f2e3dba8e` |
| `WAHA_API_KEY_PLAIN` | `HpSLMn77NYwsATcNrWZptvoGNJnsrfQD6pC4UDs2beSEeZAEaACaAPYyyo5tRmdt` |
| `WAHA_DASHBOARD_USERNAME` | `admin` |
| `WAHA_DASHBOARD_PASSWORD` | `86b20c3028014f5581f0e4c624ef1888` |
| `WHATSAPP_SWAGGER_USERNAME` | `admin` |
| `WHATSAPP_SWAGGER_PASSWORD` | `86b20c3028014f5581f0e4c624ef1888` |
| Postgres | `postgres://waha:e4c3d1a4cb2941327f380450456@postgres:5432/waha?sslmode=disable` |

### docker-03 (worker)

Sem dashboard/swagger habilitados — apenas API key:

| Variável | Valor |
|---|---|
| `WAHA_API_KEY` | `bce168adbd94e05f3c5ef43f813202bb197e243892b38e1e6bb9c42f2e3dba8e` |
| `WAHA_API_KEY_PLAIN` | `HpSLMn77NYwsATcNrWZptvoGNJnsrfQD6pC4UDs2beSEeZAEaACaAPYyyo5tRmdt` |
| Postgres | `postgres://waha:e4c3d1a4cb2941327f380450456@172.16.0.166:5432/waha?sslmode=disable` |
| Redis | `redis://172.16.0.166:6379` |

> As credenciais de API key e dashboard/swagger são as mesmas nos 3 hosts.

## Configuração por ambiente

### docker-01 (testes)

| Item | Valor |
|---|---|
| `ENGINE` | `WEBJS` |
| `MEDIA_STORAGE` | `POSTGRESQL` |
| `SESSIONS` | `POSTGRESQL` |
| `WAHA_DASHBOARD_ENABLED` | `True` |
| `WHATSAPP_SWAGGER_ENABLED` | `True` |
| `WHATSAPP_SWAGGER_CONFIG_ADVANCED` | `true` |

Postgres roda na própria stack (serviço `postgres`, resolvido internamente
pelo Docker Compose).

### docker-02 (produção)

| Item | Valor |
|---|---|
| `ENGINE` | `GOWS` |
| `MEDIA_STORAGE` | `POSTGRESQL` |
| `SESSIONS_PATH` | `/app/.sessions` |
| `WAHA_DASHBOARD_ENABLED` | `True` |
| `WHATSAPP_SWAGGER_ENABLED` | `True` |
| `WHATSAPP_SWAGGER_CONFIG_ADVANCED` | `true` |

Postgres roda na própria stack (serviço `postgres`, resolvido internamente
pelo Docker Compose). Também roda Redis local, nesta mesma stack — usado
pelo processo principal e reaproveitado pelo worker do `docker-03`.

### docker-03 (worker)

| Item | Valor |
|---|---|
| `ENGINE` | `NOWEB` |
| `MEDIA_STORAGE` | `POSTGRESQL` |
| `SESSIONS_PATH` | `/app/.sessions` |
| `WAHA_DASHBOARD_ENABLED` | `False` |
| `WHATSAPP_SWAGGER_ENABLED` | `False` |

> **Nota arquitetural**: `docker-03` não possui Postgres/Redis próprios —
> conecta remotamente aos serviços do `docker-02` (`172.16.0.166`). Ou seja,
> produção é um ambiente distribuído entre os dois hosts: `docker-02`
> concentra a instância principal + storage (Postgres/Redis), e `docker-03`
> roda apenas um worker adicional que consome esse mesmo storage.

## Histórico

Substituiu a versão *community* do Waha (testada inicialmente em
`docker-01`, atualmente desativada).
