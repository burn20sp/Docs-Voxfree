# Firewall / HAProxy

## Índice

- [Visão geral](#visão-geral)
- [Conceitos e componentes do plugin](#conceitos-e-componentes-do-plugin)
- [Fluxo lógico geral (dependências)](#fluxo-lógico-geral-dependências)
- [Aplicação principal (APP)](./firewall-haproxy-app.md)
- [Serviço de WebSocket](./firewall-haproxy-websocket.md)
- [Serviços dedicados a clientes](./firewall-haproxy-clientes.md)
- [Passo a passo: cadastrar um novo serviço](#passo-a-passo-cadastrar-um-novo-serviço)
- [Monitoramento e manutenção](#monitoramento-e-manutenção)

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

> O cadastro do domínio (DNS) e a emissão de certificados são tratados fora
> do HAProxy (DNS do domínio no provedor; certificado via plugin **ACME**
> do OPNsense, quando o domínio não é coberto pelos wildcards).

Existem dois cenários, dependendo se o serviço vai usar o pool padrão do
APP ou precisa de servidor(es) interno(s) dedicado(s):

### Cenário A — domínio do cliente, sem servidor dedicado (usa o pool padrão do APP)

Único passo: adicione o subdomínio no campo **Certificates** do Public
Service padrão `frontend-app-default` (`Services > Haproxy > Settings >
Virtual Services > Public Services`). Sem Rule dedicada, o tráfego já cai
no backend padrão (`backend-app-default`).

### Cenário B — servidor(es) interno(s) dedicado(s)

Ex.: um novo cliente nos moldes do
[agil-torpedo](./firewall-haproxy-clientes.md).

0. **Certificado** (se aplicável): se o domínio **não** for subdomínio de
   `voxfree.com` ou `voxfree.com.br` (já cobertos pelos wildcards), gere o
   certificado no plugin **ACME** do OPNsense antes de continuar.
1. **Real Server(s)** — `Services > Haproxy > Settings > Real Servers`.
   Campos principais: Nome, IP e Porta. Se for usar health-check em porta
   diferente da porta principal do serviço, abra **Advanced** e configure a
   porta em **Port to check**.
2. **Health Monitor** (opcional) — `Services > Haproxy > Settings > Rules &
   Checks > Health Monitor`. Só necessário se o serviço for usar
   health-check; caso contrário, pule para o próximo passo.
3. **Backend Pool** — `Services > Haproxy > Settings > Virtual Services >
   Backend Pools`. Indique o(s) Real Server(s) do passo 1 em **Servers**.
   Se o serviço usar health-check, marque **Enable Health Checking** e
   preencha os campos exibidos.
4. **Condition** — `Services > Haproxy > Settings > Rules & Checks >
   Conditions`. Define o domínio que originará a regra: em **Condition
   type**, selecione `hdr`, e informe o domínio em **Host String**.
5. **Rule** — `Services > Haproxy > Settings > Rules & Checks > Rules`. Em
   **Select conditions**, escolha a Condition do passo 4. Em **Type**,
   selecione `Use specified Backend Pool`, e em **Use backend pool**,
   selecione o Backend Pool do passo 3.
6. **Vincular ao Public Service** `frontend-app-default`: além do
   certificado (mesmo campo **Certificates** do Cenário A), adicione a Rule
   criada no passo 5 no campo **Select Rules**.

Com isso o encaminhamento já deve ocorrer corretamente.

## Monitoramento e manutenção

- **Página de estatísticas**: <http://172.16.0.1:8822/haproxy?stats> —
  permite acompanhar as métricas dos serviços/backends configurados.
- **Manutenção de Real Server**: para desativar temporariamente o envio de
  tráfego a um Real Server específico (sem removê-lo da configuração),
  acesse `Services > Haproxy > Maintenance`.
