# Firewall/HAProxy — Serviço de WebSocket

> Parte de [Firewall/HAProxy](./firewall-haproxy.md). Consulte lá os
> conceitos gerais (Public Service, Backend, Rule, Condition, Health
> Monitor) e o fluxo lógico de dependências.

## Public Service

> **A confirmar**: não ficou claro se o WebSocket usa um Public Service
> próprio (padrão `frontend-{app}-default`/`redirect`) ou se o tráfego
> entra pelo mesmo `frontend-app-default` (443) do APP e é desviado para o
> backend do socket via a Rule abaixo, com base no domínio. A segunda
> hipótese é a mais provável dado que o roteamento é feito por Condition de
> domínio (`hdr`), mas precisa ser confirmada.

## Backend

- **`backend-http-socket`**: backend do WebSocket.
- **Modo Active/Backup** (diferente do APP, que balanceia entre 4
  RealServers): apenas o `http-socket-01` fica ativo; o `http-socket-02`
  atua como **backup**, assumindo somente se o primário falhar no
  health-check.

### Health Monitor

| Campo | Valor |
|---|---|
| **Nome** | `Health App Socket` |
| **Verificação** | `HTTP GET /health` |
| **Esperado** | regex `"status":\s*"ok"` no corpo da resposta |
| **Porta do health-check** | 8284 *(a 8283 é dedicada ao protocolo WebSocket e não responde a HTTP comum, por isso o health-check é feito em porta separada)* |

## RealServers

| RealServer | IP | Porta | Papel |
|---|---|---|---|
| `http-socket-01` | 172.17.0.101 | 8283 | Ativo (primário) |
| `http-socket-02` | 172.17.0.102 | 8283 | Backup |

> ⚠️ **Alerta**: estes hosts estão na faixa `172.17.0.X`, enquanto o
> restante da documentação (firewall e RealServers do APP) usa
> `172.16.0.X`. Confirmar se é uma segmentação de rede intencional (ex.:
> VLAN/sub-rede dedicada ao WebSocket) ou inconsistência a corrigir.

## Rules

| Rule | Vinculada a | Condition | Critério | Comportamento |
|---|---|---|---|---|
| `rule_backend_socket` | `backend-http-socket` | *(nome não informado)* | Domínio (`hdr`, Host header) = `websocketapp.voxfree.com.br` | Direciona requisições desse domínio para o `backend-http-socket` |
