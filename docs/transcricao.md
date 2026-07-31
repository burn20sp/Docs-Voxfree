[← README](../README.md)

# Serviço de Transcrição (FastWhisper)

Serviço de transcrição de áudio (speech-to-text). Usado pelo módulo de WhatsApp do Omnichannel para transcrever
áudios enviados via chat.

## Visão geral

| Campo               | Valor                                                         |
| ------------------- | ------------------------------------------------------------- |
| **Função**          | Transcrição de áudio (STT)                                    |
| **Host**            | `transcricao` (172.16.0.120)                                  |
| **Porta**           | 8001                                                          |
| **Linguagem**       | Python                                                        |
| **Diretório base**  | `/opt/FastWhisper`                                            |
| **Serviço systemd** | `fast-whisper.service`                                        |
| **Consumidores**    | Módulo WhatsApp (Omnichannel) — transcrição de áudios de chat |

## Configuração do modelo

| Item                      | Valor                                 |
| ------------------------- | ------------------------------------- |
| **Model**                 | `small`                               |
| **Device**                | `cpu`                                 |
| **Compute type**          | `int8`                                |
| **Engine de diarization** | `huggingface/transformers` (pyannote) |

> **Nota**: esses parâmetros estão _hardcoded_ no código-fonte — não há
> `.env` ou config externa. Para alterar model/device/compute_type é
> necessário editar o código e reiniciar o serviço.

## Controle de processo

```bash
systemctl start fast-whisper.service
systemctl stop fast-whisper.service
systemctl restart fast-whisper.service
systemctl status fast-whisper.service
```

## Logs

Sem arquivo de log dedicado — consultar via journal do systemd:

```bash
journalctl -u fast-whisper.service -f
```

## Documentação da API

| Endpoint         | Descrição                         |
| ---------------- | --------------------------------- |
| `/transcribe/v1` | Endpoint principal de transcrição |
| `/docs`          | Documentação OpenAPI (Swagger)    |
| `/redoc`         | Documentação (Redocly)            |

Acesso: `http://172.16.0.120:8001/docs`

## Hugging Face (diarization)

> O token de acesso é informação sensível e não é armazenado neste
> documento.

| Item | Valor |
|---|---|
| **Token** | Configurado no arquivo de serviço do host: `/opt/FastWhisper/fast-whisper.service` |
| **Cache (`TRANSFORMERS_CACHE`)** | `/root/.cache/huggingface` |

Requisitos do token (gratuito, mas obrigatório):

- Gerar em: <https://huggingface.co/settings/tokens>
- Aceitar os termos do modelo em:
  <https://huggingface.co/pyannote/speaker-diarization-community-1>

## Repositório

Código-fonte atualmente em
[`github.com/burn20sp/FastWhisper-API-Python`](https://github.com/burn20sp/FastWhisper-API-Python)
(criado antes de existir a organização Voxfree no GitHub).

> **Pendente**: o projeto será compartilhado com Cadú e Léo para que façam o clone

## Notas operacionais

- **Configuração hardcoded**: model, device e compute_type exigem alteração
  de código (não são parametrizáveis via ambiente).
- **Sem redundância**: host único (`transcricao`), sem HA — indisponibilidade
  do host interrompe a transcrição para o módulo WhatsApp do Omnichannel.
- **Repositório pendente de compartilhamento** com Cadú e Léo (ver seção
  acima).
