# Firewall/HAProxy — Serviços dedicados a clientes

> Parte de [Firewall/HAProxy](./firewall-haproxy.md). Consulte lá os
> conceitos gerais (Public Service, Backend, Rule, Condition, Health
> Monitor), o fluxo lógico de dependências e o registro obrigatório de
> domínio/Rule no Public Service.

Cenário similar ao de produção (APP/WebSocket), porém com domínio
específico de um cliente. Exemplo representativo: **agil-torpedo**
(cliente Agil Telecom).

## Public Service

Também usa o `frontend-app-default` (443) — não há Public Service dedicado
por cliente. Diferença em relação ao APP/WebSocket: o domínio não está
sob os wildcards `*.voxfree.com`/`*.voxfree.com.br`, é um domínio do
próprio cliente (`agiltelecom.com.br`), então precisa de certificado ACME
próprio adicionado ao `frontend-app-default` — ver
[registro obrigatório no Public Service](./firewall-haproxy.md#registro-obrigatório-de-domínio-e-rule-no-public-service).

## Backend

- **`backend-agil-torpedo`**: backend dedicado ao cliente.
- **Sem Health Monitor** — diferente do APP e do WebSocket, este backend
  não tem health-check configurado.

## RealServers

| RealServer | IP | Porta |
|---|---|---|
| `agil-torpedo` | 172.16.0.206 | 80 |

## Rules

| Rule | Vinculada a | Condition | Critério | Comportamento |
|---|---|---|---|---|
| `rule_backend_agil_torpedo` | `backend-agil-torpedo` | `acl_agil-torpedo` | Domínio (`hdr`, Host header) = `torpedo.agiltelecom.com.br` | Direciona requisições desse domínio para o `backend-agil-torpedo` |

> Lembrete: além de cadastrar a Rule, ela precisa ser adicionada em
> **Select Rules** do `frontend-app-default`, e o certificado do domínio
> `torpedo.agiltelecom.com.br` precisa estar no campo **Certificates** do
> mesmo Public Service — do contrário a Condition nunca é avaliada.
