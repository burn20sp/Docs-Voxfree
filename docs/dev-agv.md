[← README](../README.md)

# dev-agv (Ambiente de Desenvolvimento)

Host destinado ao uso pessoal do responsável, para testes e
desenvolvimento: mantinha o VSCode conectado remotamente, sendo usado para
testes iniciais de novas aplicações antes de migrar para hosts dedicados,
além de testes de pacotes, versões e atualizações antes de aplicar em
hosts de produção.

## Visão geral

| Campo | Valor |
|---|---|
| **Host** | `dev-agv` (172.17.0.117) |
| **Função** | Ambiente de desenvolvimento/testes (uso pessoal) |
| **Projetos** | Diversos, em andamento/concluídos/arquivados, distribuídos em `/root` e `/opt` |
| **Exceção em produção** | NLP (`nlp-v2`) — ver seção abaixo |

## Diretórios de projetos

- `/root` — projetos diversos (sem detalhamento individual)
- `/opt` — projetos diversos (sem detalhamento individual), incluindo a
  stack Docker do NLP (`dev-stack`)

## NLP (nlp-v2) — único serviço em produção neste host

> Apesar do host ser de desenvolvimento/testes, este serviço específico
> acabou permanecendo em produção aqui, mesmo tendo sido inicialmente
> disponibilizado apenas para testes.

| Item | Valor |
|---|---|
| **Acesso** | Porta 4000, via proxy Traefik |
| **Diretório da stack (compose)** | `/opt/dev-stack` |
| **Diretório do serviço** | `/opt/dev-stack/services/nlp-v2` |
| **Banco de dados (modelos)** | 172.16.0.94 (banco principal) |
| **Redis** | Local, neste host |
| **Repositório** | `https://github.com/burn20sp/NLP-v2` (pessoal) |

> **Pendente**: repositório hospedado no GitHub pessoal do responsável —
> necessário compartilhar acesso (mesmo padrão dos demais projetos internos
> sem repositório na organização).

## Notas operacionais

- **Apenas o NLP (`nlp-v2`) está em produção** neste host; os demais
  projetos em `/root` e `/opt` são de desenvolvimento/teste, sem SLA de
  disponibilidade.
- **Cuidado ao reiniciar/desligar o host** para manutenção — isso impacta
  diretamente a produção do NLP.
- Recomenda-se, no futuro, migrar o NLP para um host de produção dedicado,
  já que hoje roda em um host cuja função original é de desenvolvimento.
