# Servidores Web

## Visão geral

| Campo | Valor |
|---|---|
| **Função** | Hosts de aplicação — executam a aplicação PHP/nginx que atende via HAProxy |
| **Quantidade** | 4 hosts |
| **Plataforma** | PHP 7.4.33 + nginx + php-fpm |
| **Storage** | Compartilhado via NFS (sincronismo entre hosts) |

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

Arquivo `/etc/fstab` em cada host (exemplo):

```
172.16.0.91:/data /var/www/html nfs defaults 0 0
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
systemctl restart php-fpm
```

Ou ambos:
```bash
systemctl restart nginx php-fpm
```

## Referências

- Documentação oficial nginx: <https://nginx.org/en/docs/>
- Documentação oficial PHP-FPM: <https://www.php.net/manual/en/install.fpm.php>
- Documentação NFS: <https://wiki.linux-nfs.org/>
