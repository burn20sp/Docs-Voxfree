# Firewall/HAProxy — Serviços dedicados a clientes

> Parte de [Firewall/HAProxy](./firewall-haproxy.md). Consulte lá os
> conceitos gerais (Public Service, Backend, Rule, Condition, Health
> Monitor), o fluxo lógico de dependências e o registro obrigatório de
> domínio/Rule no Public Service.

Cenário similar ao de produção (APP/WebSocket), porém com domínio
específico de um cliente. Exemplo representativo: **agil-torpedo**
(cliente Agil Telecom).

### Quando este cadastro completo é necessário

Este serviço tem **dois** aspectos dedicados ao cliente, não só o domínio:

1. Domínio próprio (`torpedo.agiltelecom.com.br`, fora dos wildcards
   voxfree).
2. **Ambiente interno próprio** — servidor(es) dedicado(s), diferente do
   pool padrão do APP.

Isso importa porque os dois aspectos são independentes: se o cliente
tivesse só domínio próprio, mas usasse o mesmo grupo de servidores do APP,
bastaria adicionar o certificado do domínio ao `Certificates` do
`frontend-app-default` — sem Rule, sem Condition, sem Backend/RealServer
dedicados. Nesse caso, por não haver Rule específica pra esse domínio, a
requisição cairia direto no catch-all (`backend-app-default`).

A sequência completa (Backend + RealServer + Rule + Condition dedicados,
como abaixo) só é necessária quando o cliente tem **domínio próprio E
servidores internos dedicados** — como é o caso do agil-torpedo.

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
