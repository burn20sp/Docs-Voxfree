# Documentação Voxfree

Documentação consultiva das aplicações e serviços de infraestrutura, para
apoiar manutenção, rotina de start/stop, deploy e processos operacionais.

## Repositório da Aplicação

- **GitHub**: <https://github.com/Voxfree/APP>
- **Branches principais**:
  - `main` — deploy automático para **produção** (nunca editar diretamente)
  - `homolog` — deploy automático para **homologação** (nunca editar diretamente)
  - Branches adicionais — criadas para diferentes etapas de atualizações e correções

## Aplicações

| Nome | Descrição | IP |
|---|---|---|
| [Firewall/HAProxy](./firewall-haproxy.md) | Responsável pelo roteamento HTTP baseado em domínio | 172.16.0.1 |
| [Servidores Web](./webservers.md) | Hosts de aplicação (PHP/nginx) — 4 hosts balanceados | 172.16.0.82-85 |
| [Web Homologação](./webservers-homolog.md) | Ambiente de teste (1 host) | 172.16.0.89 |
| [Rundeck](./rundeck.md) | Servidor de CRON (agendamento e execução de tarefas) | 172.16.0.165 |
