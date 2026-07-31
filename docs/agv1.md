[← README](../README.md)

# Agente Virtual (AGV) — Piloto

Projeto piloto de Agente Virtual, baseado em recursos CPqD
([ASR](./cpqd-asr.md) + [TTS](./cpqd-tts.md)) e Asterisk.

## Visão geral

| Campo | Valor |
|---|---|
| **Host** | `agv1` (172.16.0.123) |
| **Status** | Piloto |
| **Dependências** | [ASR (CPqD)](./cpqd-asr.md), [TTS (CPqD)](./cpqd-tts.md) |
| **Asterisk** | 18.24.3 |

## Asterisk

| Item | Valor |
|---|---|
| **Versão** | 18.24.3 |
| **Serviço systemd** | `asterisk` |
| **Contextos auxiliares** | `/etc/asterisk/extensions_apps`, `/etc/asterisk/extensions_utils` |
| **Diretório de IVRs** | `/etc/asterisk/agv/` |

## Micro-aplicações (dev-stack)

| Item | Valor |
|---|---|
| **Diretório base** | `/opt/dev-stack/services` |
| **Linguagem** | Node.js / TypeScript |
| **Gerenciamento** | Docker (exceto `audio-service`, ver nota abaixo) |

| Aplicação | Porta | Descrição |
|---|---|---|
| `audio-service` | 3010 | Servidor de áudios centralizado para os Asterisk de IVR |
| `call-event` (AGI-Server) | 4576 | Coleta metadados das chamadas (métricas de TTS, AST, tracking do fluxo, etc.) |
| `date-service` | 3031 | API utilitária de datas, usada nas IVRs |
| `grammar-service` | 9011 | Servidor de gramáticas, acessado pelo ASR com instruções de processamento semântico para interpretação de respostas |
| `cache-clean-app` | — | Mini-app de manutenção do media-cache do host Asterisk |

> **`audio-service` via systemd (temporário)**: para fins de teste, está
> rodando via systemd (`audio-service.service`) em vez de Docker, como as
> demais. Migração para Docker prevista quando os testes forem concluídos.

## Controle de processo

```bash
# Asterisk
systemctl status asterisk

# audio-service (systemd, modo de teste)
systemctl status audio-service.service
```

As demais micro-aplicações (`call-event`, `date-service`, `grammar-service`,
`cache-clean-app`) são gerenciadas via Docker, cada uma em seu diretório
dentro de `/opt/dev-stack/services/`.

## Repositório / Backup

> **Atenção**: essas micro-aplicações **não possuem repositório de código
> versionado** — o código existe apenas neste host. É recomendável
> implementar uma rotina de backup do diretório `/opt/dev-stack`.

## Notas operacionais

- **Piloto**: projeto ainda em fase de piloto/testes.
- **`audio-service` via systemd é temporário** — ver nota na seção de
  micro-aplicações.
- **Sem versionamento**: risco de perda de código sem backup (ver seção
  Repositório/Backup acima).
