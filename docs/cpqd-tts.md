[← README](../README.md)

# Serviço de TTS (CPqD)

Síntese de voz (TTS), fornecida pela CPqD. Dois serviços rodam no mesmo
host: o **TTS-Server** (síntese de voz) e o **MRCP-Server**, responsável
por prover a conexão entre o Asterisk e o TTS-Server (protocolo MRCP) —
mesmo padrão do [ASR (CPqD)](./cpqd-asr.md).

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Síntese de voz (TTS) |
| **Fornecedor** | CPqD |
| **Host** | `cpqd-tts` (172.16.0.119) |
| **Serviços** | TTS-Server + MRCP-Server |
| **Consumidor** | Asterisk — [Agente Virtual / agv1](./agv1.md) (via MRCP) |

## Serviços

### TTS-Server

| Item | Valor |
|---|---|
| **Diretório (engine)** | `/opt/cpqd/tts/engine` |
| **Diretório (server)** | `/opt/cpqd/tts/server` |
| **Serviço systemd** | `tts-server.service` |
| **Log (server)** | `/var/log/cpqd/tts/server` |
| **Log (engine)** | `/opt/cpqd/tts/engine/tts.log` |

### MRCP-Server

| Item | Valor |
|---|---|
| **Diretório base** | `/opt/cpqd/mrcp` |
| **Serviço systemd** | `tts-mrcp-server.service` |
| **Log** | `/opt/cpqd/mrcp/server/log` |
| **Função** | Ponte entre Asterisk e TTS-Server (protocolo MRCP) |

> O MRCP-Server depende do TTS-Server para funcionar — sem o TTS-Server
> ativo, não há síntese de voz a entregar via MRCP.

## Controle de processo

```bash
# TTS-Server
systemctl start tts-server.service
systemctl stop tts-server.service
systemctl restart tts-server.service
systemctl status tts-server.service

# MRCP-Server
systemctl start tts-mrcp-server.service
systemctl stop tts-mrcp-server.service
systemctl restart tts-mrcp-server.service
systemctl status tts-mrcp-server.service
```

## Logs

```bash
# TTS-Server
tail -f /var/log/cpqd/tts/server/*
tail -f /opt/cpqd/tts/engine/tts.log

# MRCP-Server
tail -f /opt/cpqd/mrcp/server/log/*
```

## Instaladores

Arquivos dos instaladores utilizados: `/opt/cpqd/tts`

## Licença

> A chave de licença é informação sensível/comercial (contrato pago com a
> CPqD) e não é armazenada neste documento.

| Item | Valor |
|---|---|
| **Chave de licença** | Ver arquivo de configuração no host (abaixo); se necessário, solicitar via suporte/representante comercial da CPqD |
| **Total de licenças** | 1 |
| **Arquivo de configuração** | `/opt/cpqd/tts/engine/tts.conf` |

## Acesso ao host

Login inicial é feito com o usuário `user` (sem privilégios administrativos).
Para tarefas administrativas, seguir usando `sudo` ou trocar para root
(`su -`).

## Notas operacionais

- **Licença única**: apenas 1 licença disponível — atenção à capacidade de
  uso simultâneo antes de expandir consumidores.
- **Dependência entre serviços**: reiniciar o TTS-Server pode exigir
  atenção ao MRCP-Server (verificar se a conexão é restabelecida
  automaticamente ou se precisa de restart manual).
- **Ver também**: [ASR (CPqD)](./cpqd-asr.md) — mesmo fornecedor, mesmo
  padrão de arquitetura (host dedicado + MRCP-Server), host separado.
- **Ver também**: [TTS Próprio (Piper)](./vf-tts.md) — projeto interno em
  fase de testes, avaliado como possível alternativa a este serviço.
