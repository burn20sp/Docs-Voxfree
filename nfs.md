# Servidor NFS com Cluster (Pacemaker/Corosync)

Storage compartilhado para [Servidores Web](./webservers.md) e
[Web Homologação](./webservers-homolog.md), gerenciado por cluster de alta
disponibilidade.

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Storage centralizado (NFS) com HA |
| **Hosts** | 2 nós (http-nfs-1, http-nfs-2) |
| **Cluster** | Pacemaker + Corosync |
| **VIP (Virtual IP)** | 172.16.0.91 |
| **Disco compartilhado** | VMware Shared Mode, montado em `/data` |
| **Capacidade** | 2TB |
| **Consumidores** | Servidores Web (172.16.0.82-85) e Homologação (172.16.0.89) |

## Hosts

| Hostname | IP |
|---|---|
| `http-nfs-1` | 172.16.0.92 |
| `http-nfs-2` | 172.16.0.93 |

Topologia: **ativo-passivo**. Apenas um host monta o disco compartilhado por
vez (garantia de integridade de dados). Pacemaker elege o primário e gerencia
failover automático.

## Cluster (Pacemaker/Corosync)

Serviços gerenciados (3 recursos):

### 1. VIP (Virtual IP)

| Item | Valor |
|---|---|
| **IP** | 172.16.0.91 |
| **Alocação** | Dinâmica (segue o host ativo) |
| **Função** | Ponto único de conexão para clientes (Web, Homologação) |

Clientes conectam via VIP, não aos IPs individuais dos nós. Em failover, VIP
migra automaticamente.

### 2. Storage (Disco Compartilhado)

| Item | Valor |
|---|---|
| **Device** | — |
| **Filesystem** | — |
| **Mount point** | `/data` |
| **Modo VMware** | Shared (acesso exclusivo por um host) |
| **Sincronismo** | Apenas o nó ativo tem acesso à unidade |

Pacemaker controla montagem/desmontagem dinamicamente, garantindo que apenas
um host acessa o disco por vez.

### 3. NFS Server

| Item | Valor |
|---|---|
| **Serviço** | `nfs-server` |
| **Porta** | 2049 (NFS) |
| **Gestão** | Pacemaker (inicia/encerra conforme cluster) |

Sem o disco montado, o NFS server não funciona. Pacemaker mantém o servidor
ativo apenas no nó que tem o Storage montado.

## Configuração do Cluster

| Item | Valor |
|---|---|
| **Corosync config** | `/etc/corosync/corosync.conf` |
| **Hosts file** | `/etc/hosts` (necessário registrar IPs de cada nó) |

> **Importante**: Corosync usa nome de host para comunicação. Certifique-se
> de que `/etc/hosts` tem os IPs e hostnames de ambos os nós registrados.

## Status e Monitoramento

### Verificar status do cluster

```bash
pcs status
```

Exibe:
- Qual host está eleito primário (ativo)
- Status de cada recurso (VIP, Storage, NFS Server)
- Saúde dos nós

Comando pode ser executado em **qualquer um** dos hosts.

### Logs do cluster

— (locação a ser confirmada)

## Manutenção

### Failover manual (forçar switch)

— (procedimento a ser documentado)

### Sincronização de dados

— (política/rotina a ser documentada)

## Conexão dos clientes

Clientes (Web, Homologação) conectam via **VIP `172.16.0.91`**:

```bash
# Em /etc/fstab dos servidores web/homologação:
172.16.0.91:/data                         /var/www/html   nfs     rw,vers=4,defaults 0 0
```

Failover transparente: se um nó cai, Pacemaker move VIP + Storage + NFS para
o outro nó, e clientes continuam acessando `/var/www/html` via VIP sem
interrupção.

## Questões em aberto

Informações a serem completadas:
- [ ] Device exato do disco compartilhado
- [ ] Tipo de filesystem
- [ ] Espaço atual utilizado (dos 2TB)
- [ ] Versões Pacemaker/Corosync
- [ ] Locação de logs do cluster
- [ ] Procedimento de failover manual
- [ ] Rotina de sincronização/backup de dados
- [ ] Monitoramento/alertas configurados
