# Firewall/HAProxy — Serviço de WebSocket

> Parte de [Firewall/HAProxy](./firewall-haproxy.md). Consulte lá os
> conceitos gerais (Public Service, Backend, Rule, Condition, Health
> Monitor) e o fluxo lógico de dependências.

## Public Service

Não há Public Service dedicado ao WebSocket. Assim como todos os serviços
atuais, ele entra pelo mesmo `frontend-app-default` (443) — o domínio
`websocketapp.voxfree.com.br` está coberto pelo wildcard `*.voxfree.com.br`
já vinculado a esse Public Service. O desvio para o backend correto é feito
pela Rule `rule_backend_socket` abaixo, com base no Host header.

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
| `http-socket-01` | 172.16.0.101 | 8283 | Ativo (primário) |
| `http-socket-02` | 172.16.0.102 | 8283 | Backup |

## Rules

| Rule | Vinculada a | Condition | Critério | Comportamento |
|---|---|---|---|---|
| `rule_backend_socket` | `backend-http-socket` | `acl_app_socket` | Domínio (`hdr`, Host header) = `websocketapp.voxfree.com.br` | Direciona requisições desse domínio para o `backend-http-socket` |
