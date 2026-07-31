[← README](../README.md)

# TTS Próprio (Piper)

Sistema de TTS próprio da Voxfree, com vozes open-source, baseado no
[Piper](https://github.com/rhasspy/piper). Em **fase de testes de engine**
— a previsão original era usar uma voz própria personalizada, ainda não
implementada. Composto por duas aplicações: **TTS-Engine-Service** (síntese
de voz, Python/Piper) e **TTS-FastAGI-Service** (ponte entre Asterisk e o
Engine, Node.js/TypeScript, com VAD e codificação de áudio).

> **Ver também**: [TTS (CPqD)](./cpqd-tts.md) — solução de TTS atualmente
> em uso; este projeto está em fase de testes/avaliação como alternativa.

## Visão geral

| Campo          | Valor                                                       |
| -------------- | ----------------------------------------------------------- |
| **Host**       | `vf-tts` (172.16.0.121)                                     |
| **Status**     | Fase de testes de engine (voz personalizada ainda pendente) |
| **Serviços**   | TTS-Engine-Service + TTS-FastAGI-Service                    |
| **Consumidor** | Asterisk (via FastAGI)                                      |

## TTS-Engine-Service (Piper)

| Item                | Valor                                    |
| ------------------- | ---------------------------------------- |
| **Diretório base**  | `/opt/tts-engine-service`                |
| **Serviço systemd** | `tts-engine`                             |
| **Linguagem**       | Python (FastAPI + Piper)                 |
| **Porta**           | 8000                                     |
| **Repositório**     | `github.com:burn20sp/TTS-Engine-Service` |

Endpoints:

| Endpoint                 | Descrição                                              |
| ------------------------ | ------------------------------------------------------ |
| `POST /synthesize`       | Síntese única                                          |
| `POST /synthesize/batch` | Síntese em lote (paralela via pipeline Redis)          |
| `WS /synthesize/stream`  | Streaming de síntese em tempo real                     |
| `GET /voices`            | Lista vozes disponíveis/carregadas em memória          |
| `GET /docs`              | Documentação OpenAPI (Swagger)                         |
| `GET /metrics`           | Métricas Prometheus (se `TTS_PROMETHEUS_ENABLED=true`) |

Configuração (env — valores padrão em uso):

| Variável                 | Descrição                              |
| ------------------------ | -------------------------------------- |
| `TTS_REDIS_URL`          | Conexão Redis para cache de síntese    |
| `TTS_VOICES_DIR`         | Diretório dos modelos de voz (`.onnx`) |
| `TTS_PRELOAD_VOICES`     | Vozes pré-carregadas na inicialização  |
| `TTS_PROMETHEUS_ENABLED` | Habilita endpoint `/metrics`           |
| `TTS_CACHE_TTL_SECONDS`  | TTL do cache Redis (padrão 3600s)      |

**Vozes pré-carregadas atualmente**: `pt_BR-faber-medium`,
`pt_BR-edresson-low`, `pt_BR-cadu-medium`, `pt_BR-jeff-medium`.

Cache: chave `sha256(texto+voz)`, TTL 1h, payload comprimido com zstd.

## TTS-FastAGI-Service

| Item                | Valor                                                                                   |
| ------------------- | --------------------------------------------------------------------------------------- |
| **Diretório base**  | `/opt/tts-fastagi-service`                                                              |
| **Serviço systemd** | `tts-fastagi`                                                                           |
| **Linguagem**       | Node.js / TypeScript                                                                    |
| **Função**          | Ponte entre Asterisk (FastAGI) e o TTS-Engine-Service; trata VAD e codificação de áudio |
| **Repositório**     | `github.com:burn20sp/TTS-FastAGI-Service`                                               |

Portas (valores padrão em uso):

| Item              | Valor | Descrição                                                                                  |
| ----------------- | ----- | ------------------------------------------------------------------------------------------ |
| `FASTAGI_PORT`    | 4575  | Porta que o Asterisk conecta (`agi://host:4575`)                                           |
| `AUDIO_HTTP_PORT` | 4574  | Servidor HTTP que entrega os `.wav` gerados (Playback via URL, cache ETag/304 no Asterisk) |

Fluxo de uma requisição:

1. Asterisk chama o AGI com texto + voz (via dialplan).
2. Hash = `sha256(texto+voz)`.
3. **Cache hit** (Redis) → Playback via URL
   (`http://PUBLIC_HOST:PORT/tts-agi/{hash}.wav`, <20ms).
4. **Cache miss** → chama `TTS-Engine-Service` (`/synthesize`) → salva o
   `.wav` localmente → Playback via URL.
5. Cleanup automático: LRU em disco (máx. 10GB) e TTL Redis de 24h.

Configuração (env — valores padrão em uso): `PIPER_URL` (URL do
TTS-Engine-Service), `REDIS_URL`, `CACHE_DIR`, `MAX_DISK_CACHE_GB`,
`REDIS_TTL_HOURS`, `FASTAGI_PORT`, `FASTAGI_HOST`, `PUBLIC_HOST`,
`AUDIO_HTTP_PORT`, `AUDIO_HTTP_HOST`, `AUDIO_PATH_PREFIX`,
`AUDIO_HTTP_MAX_AGE_SECONDS`.

Métricas Prometheus: `agi_requests_total`, `agi_cache_hits_total`,
`agi_cache_misses_total`, `agi_latency_seconds`, `agi_api_latency_seconds`,
`agi_errors_total`.

## Redis (cache)

Rodando em Docker, iniciado via compose global:
`/opt/global/docker-compose.yml`. Usado pelos dois serviços — cache de
síntese no Engine e verificação rápida de hash no FastAGI.

## Controle de processo

```bash
systemctl status tts-engine
systemctl status tts-fastagi
```

(`start`/`stop`/`restart` análogos, um serviço por vez.)

## Repositórios

> **Pendente**: os repositórios estão hospedados no GitHub pessoal do
> responsável, por indisponibilidade do Git da organização no momento da
> criação do projeto. Serão compartilhados com **Cadú** e **Léo** para que
> possam clonar — sem subida/migração para a organização Voxfree por
> enquanto.

- `https://github.com/burn20sp/TTS-Engine-Service`
- `https://github.com/burn20sp/TTS-FastAGI-Service`

## Notas operacionais

- **Fase de testes**: a previsão original era usar uma voz própria
  personalizada; atualmente em testes com vozes open-source do Piper.
- **Dependência entre serviços**: o TTS-FastAGI-Service depende do
  TTS-Engine-Service (via `PIPER_URL`) e do Redis — indisponibilidade de
  qualquer um interrompe o fluxo de TTS no Asterisk.
- **Cache em duas camadas**: Redis (TTL 24h no FastAGI, 1h no Engine) +
  disco local no FastAGI (LRU, máx. 10GB).
- **Ver também**: [TTS — Treinamento de Vozes](./tts-training.md) — projeto
  (interrompido/em pausa) de treinamento da voz personalizada planejada
  para este serviço.
