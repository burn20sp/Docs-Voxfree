# Firewall/HAProxy — Aplicação principal (APP)

> Parte de [Firewall/HAProxy](./firewall-haproxy.md). Consulte lá os
> conceitos gerais (Public Service, Backend, Rule, Condition, Health
> Monitor) e o fluxo lógico de dependências.

Referência interna: **APP** — a aplicação SaaS principal.

## Public Services

| Nome | Porta | Função | Domínio(s) associado(s) |
|---|---|---|---|
| `frontend-app-default` | 443 (HTTPS) | Entrada HTTPS padrão — **compartilhada por todos os serviços atuais**, não exclusiva do APP (ex.: o WebSocket também entra por aqui) | Qualquer subdomínio coberto pelos dois wildcards padrão (`*.voxfree.com` e `*.voxfree.com.br`), ou domínio ACME configurado no Public Service, desde que sem regra explícita para outro destino |
| `frontend-app-redirect` | 80 (HTTP) | Apenas força o encaminhamento para a 443 | — |

## Backend

- **`backend-app-default`**: backend padrão do APP. Referencia os
  RealServers abaixo; o tráfego é balanceado entre eles.
- **Comportamento padrão (catch-all)**: todo tráfego que **não** se encaixa
  em nenhuma regra de domínio/path explícita cai neste backend — ou seja,
  `backend-app-default` funciona como destino default do HAProxy para o
  APP.

### Health Monitor

| Campo | Valor |
|---|---|
| **Nome** | `Health App PHP` |
| **Verificação** | `GET /health/index.php` |
| **Esperado** | `200 OK` |

## RealServers

Tráfego balanceado entre os 4 hosts, todos na porta **80**:

| RealServer | IP | Porta |
|---|---|---|
| `http-app-82` | 172.16.0.82 | 80 |
| `http-app-83` | 172.16.0.83 | 80 |
| `http-app-84` | 172.16.0.84 | 80 |
| `http-app-85` | 172.16.0.85 | 80 |

## Rules

| Rule | Vinculada a | Condition | Critério | Comportamento |
|---|---|---|---|---|
| `rule_path_gravacoes` | `backend-app-default` | `acl_app_gravacoes` | Path Prefix (`path_beg`): `/gravacoes/` | Redireciona path `/gravacoes/` → `/gravacoesHD/` |
