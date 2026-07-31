[← README](../README.md)

# Servidores Web — Homologação

Ambiente de teste antes de deploy em produção. Estrutura e configuração idênticas
aos [Servidores Web de produção](./webservers.md), mas recebem deploy automático
da branch `homolog` do repositório.

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Host de aplicação — ambiente de homologação (teste) |
| **Quantidade** | 1 host |
| **Plataforma** | PHP 7.4.33 + nginx + php-fpm |
| **IP** | 172.16.0.89 |
| **Branch** | `homolog` (deploy automático) |

> **Branch protection**: a branch `homolog` nunca deve ser editada diretamente.
> Mudanças são integradas via pull requests da branch de desenvolvimento.

## Repositório

- **GitHub**: <https://github.com/Voxfree/APP>
- **Branch de homologação**: `homolog` (deploy automático, nunca editar diretamente)
- **Branches adicionais**: criadas para diferentes etapas de atualizações e correções
- **Workflow**: `.github/workflows/deploy-homolog.yml`

## Host

| Hostname | IP |
|---|---|
| `http-dev-89` | 172.16.0.89 |

## Configuração

- **Runtime**: PHP 7.4.33
- **Servidor Web**: nginx
- **Processador PHP**: php-fpm
- **Storage**: Local (não utiliza NFS externo)
- **Health-check**: `http://<IP>/health`

## Storage local

Armazenamento de código e arquivos da aplicação é **local** no host
`http-dev-89`, diferente do ambiente de produção que utiliza NFS
compartilhado.

## Sessions (Redis)

As sessões do PHP são registradas em um servidor Redis centralizado:

| Item | Valor |
|---|---|
| **Redis Server** | 172.16.0.81 |
| **Escopo** | Compartilhado entre homologação e produção |

## Deploy automático (GitHub Actions)

A aplicação é versionada no GitHub, com deploy automático via GitHub Actions
(workflow: `.github/workflows/deploy-homolog.yml`). O processo conecta ao
host `http-dev-89` para pull/deploy de mudanças.

### Conectividade GitHub → http-dev-89

| Item | Valor |
|---|---|
| **Host destino** | http-dev-89 (172.16.0.89) |
| **Porta firewall (OPNsense)** | 8198 |
| **Porta local (SSH)** | 22 |
| **Mapeamento** | Redirecionamento: firewall 8198 → host 22 |
| **Autenticação** | Chave SSH (ed25519) |

### Chave pública (deploy)

Chave pública autorizada em `~/.ssh/authorized_keys` do usuário `root`:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBwfXqBiCFJYByk32SLI+u3getjVa9M7iA7qShx6bBLO root@http-dev-89
```
