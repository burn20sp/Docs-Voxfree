# Documentação Voxfree

Documentação consultiva das aplicações e serviços de infraestrutura, para
apoiar manutenção, rotina de start/stop, deploy e processos operacionais.

## Aplicações

| Nome | Descrição | IP |
|---|---|---|
| [Firewall/HAProxy](./firewall-haproxy.md) | Responsável pelo roteamento HTTP baseado em domínio | 172.16.0.1 |
| [Servidores Web](./webservers.md) | Hosts de aplicação (PHP/nginx) — 4 hosts balanceados | 172.16.0.82-85 |
| [Web Homologação](./webservers-homolog.md) | Ambiente de teste (1 host) | 172.16.0.89 |
| [WebSocket](./websocket.md) | Presença e roteamento de mensagens para agentes (ativo-passivo) | 172.16.0.101-102 |
| [NFS com HA](./nfs.md) | Storage centralizado (Pacemaker/Corosync, 2TB) | 172.16.0.91 (VIP) |
| [Redis](./redis.md) | Cache centralizado e sessions (PHP/WebSocket) | 172.16.0.81 |
| [Rundeck](./rundeck.md) | Servidor de CRON (agendamento e execução de tarefas) | 172.16.0.165 |
| [MongoDB](./mongodb.md) | Banco de dados NoSQL, cluster ReplicaSet (Tráfego/Gateways VoIP) | 172.16.0.125-128 |
