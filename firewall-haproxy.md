# Firewall / HAProxy

## Visão geral

| Campo | Valor |
|---|---|
| **Nome** | Firewall/HAProxy |
| **Descrição** | Responsável pelo roteamento HTTP baseado em domínio |
| **IP** | 162.16.0.1 |
| **Plataforma** | OPNsense (firewall geral) + plugin `os-haproxy` (versão 5.1) |

Este documento tem propósito consultivo: orientar a manutenção de roteamentos já
cadastrados e a criação de novos serviços no HAProxy, sem depender de
conhecimento prévio da configuração atual.

## Conceitos e componentes do plugin

- **Domínio wildcard**: domínio coringa já registrado no sistema, usado para
  cobrir múltiplos subdomínios sem certificado dedicado por subdomínio.
- **Domínios ACME**: domínios com certificado gerenciado por outro plugin
  (ACME), renovação automática.
- **RealServers**: servidores internos de destino (alguns com health-check
  ativo).
- **Backend Pools**: agrupamento de RealServers que atende a um Backend.
- **Public Services**: serviços públicos, atuando na porta 443 (HTTPS), com
  regra de redirecionamento da porta 80 para a 443.
- **Health Monitors**: verificações de saúde aplicadas aos RealServers/Pools.
- **Conditions**: critérios de avaliação (ex.: domínio requisitado) usados
  pelas Rules para decidir o roteamento.
- **Rules**: vinculam um Public Service a um Backend, usando uma Condition
  como critério de decisão.

## Fluxo lógico geral (dependências)

Sequência de uma requisição até o servidor interno:

1. Requisição chega no **Public Service** (porta 443), que tem domínio e
   certificado vinculados. Requisições na porta 80 são redirecionadas para a
   443 antes de seguir o fluxo.
2. O Public Service aplica as **Rules** cadastradas.
3. Cada Rule usa uma **Condition** (o *porquê*) para avaliar o domínio
   digitado e decidir se a regra se aplica.
4. Quando a Condition casa, a Rule direciona para um **Backend** (o *pra
   onde*).
5. O Backend distribui a requisição entre os **RealServers** do seu pool,
   respeitando o **Health Monitor** (quando configurado), e encaminha ao
   servidor interno correspondente.

```mermaid
flowchart LR
    Client([Cliente / Browser])

    subgraph Entrada
        HTTP[Porta 80]
        HTTPS[Porta 443]
        Redirect[Regra de redirect\n80 → 443]
        HTTP --> Redirect --> HTTPS
    end

    Client --> HTTP
    Client --> HTTPS

    HTTPS --> PS[Public Service\ndomínio + certificado]
    PS --> Rules[Rules]

    Rules -->|avalia| Condition[Condition\n'o porquê'\ndomínio requisitado]
    Condition -->|match| Backend[Backend / Pool\n'o pra onde']

    Backend --> HM[Health Monitor]
    Backend --> RS1[RealServer 1]
    Backend --> RS2[RealServer 2]

    RS1 --> Internal[(Servidor interno)]
    RS2 --> Internal
```

## Aplicação principal

> Pendente — aguardando descrição (domínio, Public Service, Rule, Condition,
> Backend/Pool, RealServers e Health Monitor específicos).

## Serviço de WebSocket

> Pendente — mesma estrutura de componentes da aplicação principal, porém com
> configuração dedicada.

## Serviços dedicados a clientes

> Pendente — exemplo representativo de um serviço dedicado a cliente
> (cenário similar ao de produção, com domínio específico).

## Passo a passo: cadastrar um novo serviço

> Pendente — sequência de criação/vínculo dos componentes acima na interface
> do OPNsense/HAProxy.
