[← README](../README.md)

# Serviço de ASR (CPqD)

Reconhecimento de fala (ASR) e interpretação semântica, fornecido pela
CPqD. Dois serviços rodam no mesmo host: o **ASR-Server** (transcrição e
interpretação semântica) e o **MRCP-Server**, responsável por prover a
conexão entre o Asterisk e o ASR-Server (protocolo MRCP).

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Reconhecimento de fala (ASR) e interpretação semântica |
| **Fornecedor** | CPqD |
| **Host** | `cpqd-asr` (172.16.0.118) |
| **Serviços** | ASR-Server + MRCP-Server |
| **Consumidor** | Asterisk — [Agente Virtual / agv1](./agv1.md) (via MRCP) |

## Serviços

### ASR-Server

| Item | Valor |
|---|---|
| **Diretório base** | `/opt/cpqd/asr` |
| **Serviço systemd** | `asr-server.service` |
| **Log** | `/var/log/cpqd/asr` |

### MRCP-Server

| Item | Valor |
|---|---|
| **Diretório base** | `/opt/cpqd/mrcp` |
| **Serviço systemd** | `asr-mrcp-server.service` |
| **Log** | `/opt/cpqd/mrcp/server/log` |
| **Função** | Ponte entre Asterisk e ASR-Server (protocolo MRCP) |

> O MRCP-Server depende do ASR-Server para funcionar — sem o ASR-Server
> ativo, não há reconhecimento de fala a entregar via MRCP.

## Controle de processo

```bash
# ASR-Server
systemctl start asr-server.service
systemctl stop asr-server.service
systemctl restart asr-server.service
systemctl status asr-server.service

# MRCP-Server
systemctl start asr-mrcp-server.service
systemctl stop asr-mrcp-server.service
systemctl restart asr-mrcp-server.service
systemctl status asr-mrcp-server.service
```

## Logs

```bash
# ASR-Server
tail -f /var/log/cpqd/asr/*

# MRCP-Server
tail -f /opt/cpqd/mrcp/server/log/*
```

## Instaladores

Arquivos dos instaladores utilizados: `/opt/cpqd/download`

## Licença

> A chave de licença é informação sensível/comercial (contrato pago com a
> CPqD) e não é armazenada neste documento.

| Item | Valor |
|---|---|
| **Chave de licença** | Ver arquivo de configuração no host (abaixo); se necessário, solicitar via suporte/representante comercial da CPqD |
| **Total de licenças** | 1 |
| **Arquivo de configuração** | `/opt/cpqd/asr/config/engine/engine.conf` (parâmetro `--licenseManager.licenseId`) |

## Acesso ao host

Este host **não aceita login root direto via SSH**. O acesso deve ser feito
primeiro com o usuário `user` (sem privilégios administrativos); a partir
dele, se necessário, usar `sudo` ou trocar para root (`su -`). A senha do
usuário `user` é a mesma senha padrão utilizada nos demais hosts.

## Notas operacionais

- **Licença única**: apenas 1 licença disponível — atenção à capacidade de
  uso simultâneo antes de expandir consumidores.
- **Dependência entre serviços**: reiniciar o ASR-Server pode exigir
  atenção ao MRCP-Server (verificar se a conexão é restabelecida
  automaticamente ou se precisa de restart manual).
- **Ver também**: [TTS (CPqD)](./cpqd-tts.md) — mesmo fornecedor, mesmo
  padrão de arquitetura (host dedicado + MRCP-Server), host separado.
