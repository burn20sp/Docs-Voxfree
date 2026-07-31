[← README](../README.md)

# TTS — Treinamento de Vozes (Pesquisa)

Host dedicado a pesquisa e testes de treinamento para criação de vozes
próprias, para uso futuro no [TTS Próprio (Piper)](./vf-tts.md). **Projeto
interrompido no meio do caminho, não concluído** — atualmente em fase de
*tuning* do processo de treinamento, antes de iniciar a captura definitiva
com um locutor definitivo.

## Visão geral

| Campo | Valor |
|---|---|
| **Host** | `tts-trainning` (172.16.0.124) |
| **Status** | Projeto interrompido / em pausa — fase de tuning, não concluído |
| **Recursos** | Processo pesado, sem GPU disponível (roda apenas em CPU) |
| **Estado atual do host** | **Desligado** (evitar gasto de recursos com ociosidade) |

## Ferramentas utilizadas

| Ferramenta | Função |
|---|---|
| [piper-recording-studio](https://github.com/rhasspy/piper-recording-studio) | Captura/gravação dos chunks de voz usados no treinamento |
| tensorboard-monitor | Acompanha o processo de treinamento e auxilia na escolha dos melhores chunks |

## Diretório

Arquivos do projeto em `/root/`.

## Execução

Não há processo em execução contínua (sem serviço/systemd). O treinamento
roda em Python e é executado **manualmente**, conforme necessidade.

## Notas operacionais

- **Host desligado por padrão**: ligar apenas quando for retomar o trabalho
  de tuning/treinamento, para não desperdiçar recursos computacionais.
- **Sem GPU**: o processo de treinamento é pesado e lento, rodando somente
  em CPU.
- **Objetivo**: concluído o tuning do processo, iniciar a captura de voz
  com um locutor definitivo para treinar a voz personalizada planejada
  para o [TTS Próprio (Piper)](./vf-tts.md).
