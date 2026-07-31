[← README](../README.md)

# Uptime Kuma (Monitoramento)

Aplicação de monitoramento simples de uptime/status de serviços. Instalada
recentemente e apresentada ao Cadú — **ainda em fase inicial de
implantação** (piloto/avaliação, não confirmada em produção definitiva).

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Monitoramento de uptime/status |
| **Host** | Desenvolvimento (172.16.0.117) |
| **Porta** | 3001 |
| **Diretório base** | `/root/uptime-kuma` |
| **Gerenciamento** | Docker Compose |
| **Status** | Fase inicial de implantação (piloto) |

## Acesso

| Item | URL |
|---|---|
| **Dashboard (admin)** | <http://172.16.0.117:3001/dashboard> |
| **Página de status pública** | <http://172.16.0.117:3001/status/v1> |

## Credenciais

> **Atenção**: por padrão não é recomendado documentar credenciais em texto
> plano. Estão registradas aqui a pedido, para referência operacional —
> restrinja o acesso a este arquivo/repositório e trate como informação
> sensível.

| Item | Valor |
|---|---|
| **Usuário** | `admin` |
| **Senha** | `Mudar@1357` |

## Migração planejada

Atualmente hospedado em host de **desenvolvimento** (172.16.0.117), fora do
parque de [Hosts Docker](./docker.md).

> **Pendente**: em caso de aprovação do piloto, migrar a aplicação para o
> parque de hosts Docker (`docker-01`/`02`/`03`).

## Referências

- Documentação oficial (wiki): <https://github.com/louislam/uptime-kuma/wiki>
