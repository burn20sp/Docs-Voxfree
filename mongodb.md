# MongoDB (Cluster ReplicaSet)

Banco de dados NoSQL usado por Tráfego/Gateways VoIP, em cluster ReplicaSet
de 3 nós para alta disponibilidade.

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Banco de dados (MongoDB ReplicaSet) |
| **Hosts** | 3 nós (mongodb-01, mongodb-02, mongodb-03) |
| **Replica Set** | `rs0` |
| **Versão** | 8.0.x |
| **Porta** | 27017 (padrão) |
| **Consumidores** | Tráfego/Gateways VoIP |

## Hosts

| Hostname | IP |
|---|---|
| `mongodb-01` | 172.16.0.125 |
| `mongodb-02` | 172.16.0.126 |
| `mongodb-03` | 172.16.0.128 |

Topologia: **ReplicaSet** (1 primary + 2 secondaries, eleição automática).
Não há VIP — clientes conectam informando os 3 hosts na connection string, e o
driver descobre e acompanha automaticamente qual nó é o primary (failover
transparente via protocolo do MongoDB).

## Armazenamento

Cada host possui uma unidade de disco **dedicada aos dados do MongoDB**,
separada do disco de sistema.

| Item | Valor |
|---|---|
| **Mount point** | `/data` |
| **Capacidade** | 300GB (por host) |
| **dbPath (mongod.conf)** | `/data` |

## Configuração

| Item | Valor |
|---|---|
| **Arquivo de config** | `/etc/mongod.conf` |
| **Log** | `/var/log/mongodb/mongod.log` |
| **Serviço systemd** | `mongod.service` |

## Controle de processo

Executar em cada host individualmente:

```bash
systemctl start mongod.service
systemctl stop mongod.service
systemctl restart mongod.service
systemctl status mongod.service
```

> **Atenção**: parar o `mongod` de um nó não interrompe o cluster (os outros
> 2 nós mantêm o ReplicaSet funcional), mas reduz a redundância. Evitar parar
> mais de um nó por vez.

## Status do Replica Set

Conectar via `mongosh` em qualquer um dos nós e executar:

```javascript
// Status geral do replica set (qual é o primary, saúde dos nós)
rs.status()

// Configuração do replica set
rs.conf()

// Verifica se o nó atual é primary ou secondary
db.hello()
```

Logs em tempo real:

```bash
tail -f /var/log/mongodb/mongod.log
```

## Acesso

### Credenciais

> **Atenção**: por padrão não é recomendado documentar credenciais em texto
> plano. Estão registradas aqui a pedido, para referência operacional —
> restrinja o acesso a este arquivo/repositório e trate como informação
> sensível.

| Item | Valor |
|---|---|
| **Usuário** | `root` |
| **Senha** | `iavoyQiZ8f4ZG2d58TFE3UbBEM3JMkLidnjK` |
| **authSource** | `admin` |

### URI de conexão

```
mongodb://root:iavoyQiZ8f4ZG2d58TFE3UbBEM3JMkLidnjK@172.16.0.125:27017,172.16.0.126:27017,172.16.0.128:27017/?authSource=admin&replicaSet=rs0&readPreference=secondaryPreferred
```

### Conexão via mongosh

```bash
mongosh "mongodb://root:iavoyQiZ8f4ZG2d58TFE3UbBEM3JMkLidnjK@172.16.0.125:27017,172.16.0.126:27017,172.16.0.128:27017/?authSource=admin&replicaSet=rs0&readPreference=secondaryPreferred"
```

## Notas operacionais

- **`readPreference=secondaryPreferred`**: leituras são direcionadas
  preferencialmente aos secondaries, aliviando carga do primary. Escritas
  sempre vão para o primary.
- **Usuário único (`root`)**: aplicações e administração compartilham a mesma
  credencial. Recomenda-se, quando possível, criar um usuário de aplicação
  com permissões restritas (`readWrite` no banco específico) em vez de usar
  `root` para conexões de aplicação.
- **Failover automático**: se o primary cair, o ReplicaSet elege um novo
  primary entre os secondaries automaticamente (maioria de votos — com 3 nós,
  tolera a queda de 1 nó sem perda de disponibilidade de escrita).
- **Backup**: não há rotina de backup documentada até o momento. Recomenda-se
  configurar `mongodump` agendado ou snapshots do disco `/data`.
