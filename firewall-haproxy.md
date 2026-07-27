# Firewall / HAProxy

## Índice

- [Visão geral](#visão-geral)
- [Conceitos e componentes do plugin](#conceitos-e-componentes-do-plugin)
- [Fluxo lógico geral (dependências)](#fluxo-lógico-geral-dependências)
- [Aplicação principal (APP)](./firewall-haproxy-app.md)
- [Serviço de WebSocket](./firewall-haproxy-websocket.md)
- [Serviços dedicados a clientes](./firewall-haproxy-clientes.md)
- [Passo a passo: cadastrar um novo serviço](#passo-a-passo-cadastrar-um-novo-serviço)

## Visão geral

| Campo | Valor |
|---|---|
| **Nome** | Firewall/HAProxy |
| **Descrição** | Responsável pelo roteamento HTTP baseado em domínio |
| **IP** | 172.16.0.1 |
| **Plataforma** | OPNsense (firewall geral) + plugin `os-haproxy` (versão 5.1) |

## Conceitos e componentes do plugin

- **Domínios wildcard**: por padrão, existem **dois** domínios wildcard
  cadastrados — `*.voxfree.com` e `*.voxfree.com.br` — cobrindo os
  subdomínios de cada TLD. Já estão registrados no sistema. **Não são
  gerenciados pelo HAProxy** — aqui são apenas referenciados/vinculados aos
  Public Services.
- **Domínios ACME**: domínios com certificado gerenciado por outro plugin
  (ACME), com renovação automática. **Também não são gerenciados pelo
  HAProxy** — igualmente, apenas referenciados/vinculados aos Public
  Services.
- **RealServers**: servidores internos de destino (alguns com health-check
  ativo).
- **Backend Pools**: agrupamento de RealServers que atende a um Backend.
- **Public Services**: serviços públicos, atuando na porta 443 (HTTPS), com
  regra de redirecionamento da porta 80 para a 443. **Padrão atual**: todos
  os serviços (APP, WebSocket, etc.) compartilham o mesmo Public Service
  `frontend-app-default` como entrada HTTPS — um Public Service dedicado só
  existe quando um serviço precisa de domínio/certificado ou comportamento
  próprio.
- **Health Monitors**: verificações de saúde aplicadas aos RealServers/Pools.
- **Conditions**: critérios de avaliação (ex.: domínio requisitado, path da
  URL) usados pelas Rules para decidir o roteamento.
- **Rules**: vinculam um Public Service a um Backend, usando uma Condition
  como critério de decisão.

### Registro obrigatório de domínio (e, se houver, Rule) no Public Service

Para que um domínio seja respondido corretamente por um Public Service
HTTPS (ex.: `frontend-app-default`):

1. **Certificates** (sempre obrigatório): o certificado do domínio
   (wildcard ou ACME) precisa ser adicionado ao campo `Certificates` do
   Public Service. Sem isso o domínio não é respondido.
2. **Select Rules** (só se houver encaminhamento dedicado): uma Rule só é
   necessária quando o domínio precisa ser encaminhado para um Backend
   diferente do padrão (ex.: servidores internos dedicados, como no caso de
   clientes dedicados). Se o domínio deve apontar para o pool padrão do APP
   (os 4 servidores web padrão), basta o certificado — o tráfego já cai no
   catch-all (`backend-app-default`) sem precisar de Rule/Condition. Quando
   uma Rule é criada, ela também precisa ser adicionada em `Select Rules`
   do Public Service, do contrário nunca é avaliada.

### Rede interna

Os IPs locais referenciados nesta documentação seguem a faixa `172.16.0.X`
(ex.: firewall em `172.16.0.1`; RealServers do APP em `172.16.0.82`–
`172.16.0.85`; RealServers do WebSocket em `172.16.0.101`–`172.16.0.102`).

## Fluxo lógico geral (dependências)

Sequência de uma requisição até o servidor interno:

1. Requisição chega no **Public Service** (porta 443), que tem domínio(s) e
   certificado(s) vinculados. Requisições na porta 80 são redirecionadas
   para a 443 antes de seguir o fluxo.
2. O Public Service aplica as **Rules** cadastradas.
3. Cada Rule usa uma **Condition** (o *porquê*) para avaliar a requisição
   (domínio, path, etc.) e decidir se a regra se aplica.
4. Quando a Condition casa, a Rule direciona para um **Backend** (o *pra
   onde*).
5. O Backend distribui a requisição entre os **RealServers** do seu pool,
   respeitando o **Health Monitor** (quando configurado), e encaminha ao
   servidor interno correspondente.

```mermaid
flowchart LR
    Client(["Cliente / Browser"])

    subgraph Entrada ["Entrada HTTP/HTTPS"]
        HTTP["Porta 80"]
        Redirect["Regra de redirect<br/>80 para 443"]
        HTTPS["Porta 443"]
        HTTP --> Redirect --> HTTPS
    end

    Client --> HTTP
    Client --> HTTPS

    HTTPS --> PS["Public Service<br/>dominio(s) + certificado(s)"]
    PS --> Rules["Rules"]

    Rules -->|avalia| Condition["Condition (o porque)<br/>dominio / path requisitado"]
    Condition -->|match| Backend["Backend / Pool (o pra onde)"]

    Backend --> HM["Health Monitor"]
    Backend --> RS1["RealServer 1"]
    Backend --> RS2["RealServer 2"]

    RS1 --> Internal[("Servidor interno")]
    RS2 --> Internal
```

## Aplicação principal (APP)

Detalhamento completo em [firewall-haproxy-app.md](./firewall-haproxy-app.md).

## Serviço de WebSocket

Detalhamento completo em [firewall-haproxy-websocket.md](./firewall-haproxy-websocket.md).

## Serviços dedicados a clientes

Detalhamento completo em [firewall-haproxy-clientes.md](./firewall-haproxy-clientes.md).

## Passo a passo: cadastrar um novo serviço

> Pendente — sequência de criação/vínculo dos componentes acima na interface
> do OPNsense/HAProxy.
