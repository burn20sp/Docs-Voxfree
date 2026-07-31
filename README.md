# Documentação Voxfree

Documentação consultiva das aplicações e serviços de infraestrutura, para
apoiar manutenção, rotina de start/stop, deploy e processos operacionais.

## Diagrama da infraestrutura web

![Fluxo HTTP: Firewall → HAProxy → Servidores Web → NFS HA / Redis](./docs/http_diagram.png)

Fluxo completo de uma requisição HTTP: [Firewall/HAProxy](./docs/firewall-haproxy.md)
→ [Servidores Web](./docs/webservers.md) (NGINX + PHP-FPM) → storage via
[NFS com HA](./docs/nfs.md) e sessions via [Redis](./docs/redis.md).

## Aplicações

| Nome                                          | Descrição                                                        | IP                |
| --------------------------------------------- | ---------------------------------------------------------------- | ----------------- |
| [Firewall/HAProxy](./docs/firewall-haproxy.md)     | Responsável pelo roteamento HTTP baseado em domínio              | 172.16.0.1        |
| [Servidores Web](./docs/webservers.md)             | Hosts de aplicação (PHP/nginx) — 4 hosts balanceados             | 172.16.0.82-85    |
| [Web Homologação](./docs/webservers-homolog.md)    | Ambiente de teste (1 host)                                       | 172.16.0.89       |
| [WebSocket](./docs/websocket.md)                   | Presença e roteamento de mensagens para agentes (ativo-passivo)  | 172.16.0.101-102  |
| [NFS com HA](./docs/nfs.md)                        | Storage centralizado (Pacemaker/Corosync, 2TB)                   | 172.16.0.91 (VIP) |
| [Redis](./docs/redis.md)                           | Cache centralizado e sessions (PHP/WebSocket)                    | 172.16.0.81       |
| [Rundeck](./docs/rundeck.md)                       | Servidor de CRON (agendamento e execução de tarefas)             | 172.16.0.165      |
| [MongoDB](./docs/mongodb.md)                       | Banco de dados NoSQL, cluster ReplicaSet (Tráfego/Gateways VoIP) | 172.16.0.125-128  |
| [Hosts Docker](./docs/docker.md)                   | Hosts independentes para aplicações diversas via Docker Compose  | 172.16.0.165-167  |
| [Waha](./docs/waha.md)                             | Plataforma de conexão para WhatsApp (testes e produção)          | 172.16.0.165-167  |
| [Transcrição (FastWhisper)](./docs/transcricao.md) | Transcrição de áudio (STT) usado pelo WhatsApp/Omnichannel       | 172.16.0.120      |
| [Uptime Kuma](./docs/uptime-kuma.md)               | Monitoramento de uptime/status (piloto, fase inicial)            | 172.16.0.117      |
| [ASR (CPqD)](./docs/cpqd-asr.md)                   | Reconhecimento de fala e interpretação semântica, usado em IVR   | 172.16.0.118      |
| [TTS (CPqD)](./docs/cpqd-tts.md)                   | Síntese de voz, usado em IVR                                     | 172.16.0.119      |
| [TTS Próprio (Piper)](./docs/vf-tts.md)             | TTS interno com vozes open-source (fase de testes)               | 172.16.0.121      |
| [TTS — Treinamento de Vozes](./docs/tts-training.md) | Pesquisa/tuning de treinamento de voz (host normalmente desligado) | 172.16.0.124      |
| [Agente Virtual (AGV) — Piloto](./docs/agv1.md)    | Piloto de Agente Virtual (Asterisk + micro-apps + ASR/TTS CPqD)  | 172.16.0.123      |
| [dev-agv (Desenvolvimento)](./docs/dev-agv.md)     | Ambiente de dev/testes; roda o NLP (nlp-v2) em produção          | 172.17.0.117      |
| [dev-claudecode (Desenvolvimento)](./docs/dev-claudecode.md) | Ambiente de dev integrado ao Claude Code (ai-gateway em andamento) | 172.16.0.16       |
