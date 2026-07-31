[← README](../README.md)

# dev-claudecode (Ambiente de Desenvolvimento — Claude Code)

Host de desenvolvimento mais recente, que deu início ao desenvolvimento
integrado com o Claude Code (IA). Um novo host se fez necessário devido aos
recursos computacionais e de disco do host originalmente destinado a esse
fim ([dev-agv](./dev-agv.md), 172.17.0.117). A programação era migrar os
projetos em andamento do host anterior para este; os novos projetos já
foram iniciados diretamente aqui.

## Visão geral

| Campo | Valor |
|---|---|
| **Host** | `dev-claudecode` (172.16.0.16) |
| **Função** | Ambiente de desenvolvimento (uso pessoal), integrado ao Claude Code |
| **Projetos** | 3, todos em `/root/dev` |
| **Credenciais** | Assinatura pessoal do Claude Code já removida do host |

## Projetos (`/root/dev`)

### ai-gateway

Baseado no [Flowise](https://flowiseai.com/), com objetivo de fornecer uma
API de recursos de IA (memória, MCP, RAG), além de um catálogo de conexões
com LLMs comerciais, para integrar ao Omnichannel/APP. Projeto nasceu da
grande demanda de personalização do Flowise (app open-source).

| Item | Valor |
|---|---|
| **Status** | Em andamento |
| **Recomendação** | Fazer backup deste projeto antes de decidir pelo desligamento do host |

### APP-Websocket

Projeto concluído, já em produção. Documentado em
[Servidores WebSocket](./websocket.md).

| Item | Valor |
|---|---|
| **Status** | Concluído |
| **Recomendação** | Diretório pode ser excluído — código já está no GitHub da Voxfree |

### CX-Customer-Experience

Clone do repositório do projeto de CS do Rodrigo. Posteriormente, a edição
e o escopo do novo projeto passaram a ser feitos diretamente no GitHub —
inicialmente no repositório do Rodrigo, depois no da Voxfree.

| Item | Valor |
|---|---|
| **Status** | Desenvolvimento migrado para o GitHub |
| **Recomendação** | Diretório pode ser excluído — desenvolvimento já ocorre diretamente no GitHub da Voxfree |

## Notas operacionais

- **Apenas o `ai-gateway` está de fato em desenvolvimento ativo** neste
  host; `APP-Websocket` e `CX-Customer-Experience` são clones locais
  obsoletos e podem ser excluídos.
- **Recomendação**: se o `ai-gateway` não for mais necessário (ou após seu
  backup), o host pode ser removido/desligado.
- **Sem credenciais pendentes**: a assinatura pessoal do Claude Code usada
  aqui já foi removida do host.
