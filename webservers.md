# Servidores Web

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Hosts de aplicação — executam a aplicação PHP/nginx que atende via HAProxy |
| **Quantidade** | 4 hosts |
| **Plataforma** | PHP 7.4.33 + nginx + php-fpm |
| **Storage** | Compartilhado via NFS (sincronismo entre hosts) |

## Repositório

- **GitHub**: <https://github.com/Voxfree/APP>
- **Branch de produção**: `main` (deploy automático, nunca editar diretamente)
- **Branches adicionais**: criadas para diferentes etapas de atualizações e correções

## Hosts

| Hostname | IP |
|---|---|
| `http-web-82` | 172.16.0.82 |
| `http-web-83` | 172.16.0.83 |
| `http-web-84` | 172.16.0.84 |
| `http-web-85` | 172.16.0.85 |

## Aplicação

- **Runtime**: PHP 7.4.33
- **Servidor Web**: nginx
- **Processador PHP**: php-fpm (FastCGI Process Manager)
- **Raiz da aplicação**: `/var/www/html`

## Storage compartilhado (NFS)

Os 4 hosts compartilham o código e arquivos da aplicação via NFS, garantindo
sincronismo:

| Item | Valor |
|---|---|
| **Servidor NFS** | 172.16.0.91 |
| **Caminho NFS** | `/data` |
| **Ponto de montagem local** | `/var/www/html` |
| **Montagem automática** | Via `/etc/fstab` |

### Configuração de montagem

Arquivo `/etc/fstab` em cada host:

```
172.16.0.91:/data                         /var/www/html   nfs     rw,vers=4,defaults 0 0
```

Sem essa montagem automática, a aplicação não funciona — o diretório
`/var/www/html` fica inacessível ao nginx/php-fpm.

## Restart/manutenção

Serviços principais (por host):

- **nginx**: servidor web
- **php-fpm**: processador de requisições PHP

Reiniciar serviço(s):
```bash
systemctl restart nginx
systemctl restart php7.4-fpm.service
```

Ou ambos:
```bash
systemctl restart nginx php7.4-fpm.service
```

## Deploy automático (GitHub Actions)

A aplicação é versionada no GitHub, com deploy automático via GitHub Actions
(workflow: `.github/workflows/deploy-prod.yml`). O processo conecta ao
primeiro servidor web (`http-web-82`) para pull/deploy de mudanças.

### Conectividade GitHub → http-web-82

| Item | Valor |
|---|---|
| **Host destino** | http-web-82 (172.16.0.82) |
| **Porta firewall (OPNsense)** | 8197 |
| **Porta local (SSH)** | 22 |
| **Mapeamento** | Redirecionamento: firewall 8197 → host 22 |
| **Autenticação** | Chave SSH (ed25519) |

### Chave pública (deploy)

Chave pública autorizada em `~/.ssh/authorized_keys` do usuário que executa
o deploy (ou root):

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJCMuPLcz9D2lXdRSFDfrLYSWLJRuUaQen8tg0sJGmCC github_deploy_key
```

## Health-check

O HAProxy realiza health-check acessando `http://<IP-HOST>/health` em cada
servidor. A resposta esperada é o **status do php-fpm** — se php-fpm está
respondendo corretamente, o host é marcado como ativo no pool de balanceamento.

