# Servidores Web — Homologação

Ambiente de teste antes de deploy em produção. Estrutura e configuração idênticas
aos [Servidores Web de produção](./webservers.md), mas recebem deploy automático
da branch `homolog` do repositório.

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Hosts de aplicação — ambiente de homologação |
| **Quantidade** | 4 hosts |
| **Plataforma** | PHP 7.4.33 + nginx + php-fpm |
| **Branch** | `homolog` (deploy automático) |

> **Branch protection**: a branch `homolog` nunca deve ser editada diretamente.
> Mudanças são integradas via pull requests da branch de desenvolvimento.

## Repositório

- **GitHub**: <https://github.com/Voxfree/APP>
- **Branch de deploy**: `homolog`
- **Workflow**: `.github/workflows/deploy-homolog.yml`

## Hosts

| Hostname | IP |
|---|---|
| `http-homolog-82` | 172.16.0.X |
| `http-homolog-83` | 172.16.0.X |
| `http-homolog-84` | 172.16.0.X |
| `http-homolog-85` | 172.16.0.X |

> IPs a serem confirmados/documentados.

## Configuração

Idêntica aos servidores de produção:
- **Runtime**: PHP 7.4.33
- **Servidor Web**: nginx
- **Processador PHP**: php-fpm
- **Storage**: NFS compartilhado (172.16.0.91:/data)
- **Health-check**: `http://<IP>/health`

## Deploy automático

Quando mudanças são merged na branch `homolog`, o GitHub Actions dispara
automaticamente o workflow de deploy, atualizando os servidores de homologação.

## Diferenças com Produção

- **Ambiente**: teste/validação antes de ir para `main`
- **Branch**: `homolog` (nunca editar diretamente) vs `main`
- **Hosts**: separados (homolog-82-85 vs web-82-85)
- **Tráfego**: volume reduzido (ambiente de teste)
