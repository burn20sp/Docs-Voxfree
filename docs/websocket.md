[← README](../README.md)

# Servidores WebSocket

Servidor WebSocket de presença e roteamento de mensagens para agentes.
Execução em ativo-passivo, com failover automático via HAProxy.

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | WebSocket de presença e roteamento de mensagens |
| **Quantidade** | 2 hosts (ativo-passivo) |
| **Plataforma** | PHP 8.5.7 |
| **Domínio** | https://websocketapp.voxfree.com.br |
| **Porta WebSocket** | 8283 |
| **Porta Health-check** | 8284 |
| **Status** | Ativo (host 01) / Backup (host 02) |

## Repositório

- **GitHub**: <https://github.com/Voxfree/APP-Websocket>
- **README**: <https://github.com/Voxfree/APP-Websocket/blob/main/README.md>

## Instalação

O processo de instalação (PHP 8.5, Composer, dependências, systemd, logrotate,
HAProxy) está descrito em detalhes no
[README oficial do repositório](https://github.com/Voxfree/APP-Websocket#instalação-debian-12-manual-nos-dois-hosts).

## Hosts

| Hostname | IP | Modo |
|---|---|---|
| `http-websocket-01` | 172.16.0.101 | Ativo |
| `http-websocket-02` | 172.16.0.102 | Backup |

Topologia: apenas o host **ativo** (01) recebe tráfego. O **backup** (02) assume
automaticamente em caso de falha do primário.

## Aplicação

- **Runtime**: PHP 8.5.7
- **Diretório**: `/opt/app-websocket`
- **Configuração**: `/etc/app-websocket/config.env`
- **Log**: `/var/log/app-websocket/app.log`

## WebSocket

| Item | Valor |
|---|---|
| **Protocolo** | wss (WebSocket Secure, via HAProxy) |
| **Porta interna** | 8283 |
| **Preservação** | Protocolo de mensagens e formato de chaves Redis — 100% compatível com clientes legados |

## Health-check

| Item | Valor |
|---|---|
| **Endpoint** | `http://<IP>:8284/health` |
| **Método** | GET |
| **Resposta** | JSON com status, instance_id, connections, timestamp |
| **Exemplo** | `{"status":"ok","instance_id":"app-webservice-01","connections":196,"timestamp":"2026-07-28T11:52:38-03:00"}` |

O HAProxy monitora esse endpoint para detectar falhas e fazer failover.

## Sessions (Redis)

| Item | Valor |
|---|---|
| **Redis Server** | 172.16.0.81 |
| **Escopo** | Compartilhado (presença, TTL, detecção de fantasmas via instance_id) |

## Logs e rotação

| Item | Valor |
|---|---|
| **Arquivo de log** | `/var/log/app-websocket/app.log` |
| **Rotação** | `/etc/logrotate.d/app-websocket` |
| **Visualizar** | `tail -f /var/log/app-websocket/app.log` |
| **Journal (systemd)** | `journalctl -u app-websocket -f` |

## Controle de processo

Serviço systemd: `app-websocket.service`

```bash
systemctl start app-websocket.service
systemctl restart app-websocket.service
systemctl stop app-websocket.service
systemctl status app-websocket.service
```

## HAProxy / OPNsense

Configurado no plugin `os-haproxy` com:
- **Backend**: `backend-http-socket` (ativo-passivo)
- **Health Monitor**: monitora porta 8284 (`/health`)
- **RealServers**: http-websocket-01 (ativo), http-websocket-02 (backup)
- **Rule**: roteia domínio `websocketapp.voxfree.com.br` para esse backend

Consulte [Firewall/HAProxy — Serviço de WebSocket](./firewall-haproxy-websocket.md)
para detalhes de roteamento.

## Comandos suportados

Protocolo WebSocket baseado em JSON (6 comandos reconhecidos). Consulte o
[README oficial](https://github.com/Voxfree/APP-Websocket#protocolo--comandos-suportados)
para especificação completa:
- `ping` / `pong`
- `send_ping`
- `subscribe`
- `call_in`
- `message`
- `broadcast`
