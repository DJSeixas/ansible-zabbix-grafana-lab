# Ansible Zabbix Grafana Lab

Projeto prático de automação com Ansible para provisionar e configurar uma stack de monitoramento com Zabbix Server, Zabbix Agent 2 e Grafana em servidores Linux.

O objetivo do projeto é praticar automação de infraestrutura usando uma estrutura organizada com playbooks, roles, variáveis, Ansible Vault e integração com API.

## Objetivo

Automatizar servidores Linux Debian, Rocky Linux e Oracle Linux com Ansible, incluindo:

* Configuração base dos servidores
* Instalação de pacotes comuns
* Instalação do Zabbix Server no Debian
* Instalação do Zabbix Agent 2 nos hosts Linux
* Cadastro dos hosts no Zabbix via API
* Instalação do Grafana
* Instalação do plugin Zabbix para Grafana
* Uso de Ansible Vault para senhas
* Separação por roles
* Execução idempotente dos playbooks

## Ambiente

| Host         | Sistema      | Função                                |
| ------------ | ------------ | ------------------------------------- |
| srv_debian01 | Debian       | Zabbix Server, Grafana e Zabbix Agent |
| srv_rocky01  | Rocky Linux  | Zabbix Agent                          |
| srv_oracle01 | Oracle Linux | Zabbix Agent                          |

## Estrutura principal

```text
ansible-lab/
├── ansible.cfg
├── inventory.ini
├── requirements.yml
├── group_vars/
├── host_vars/
├── vault/
├── playbooks/
│   ├── site.yml
│   ├── pre_check.yml
│   ├── bootstrap.yml
│   ├── monitoring.yml
│   ├── zabbix_api.yml
│   └── maintenance.yml
└── roles/
    ├── precheck/
    ├── common/
    ├── patch_management/
    ├── zabbix_server/
    ├── zabbix_agent/
    ├── zabbix_api/
    └── grafana/
```

## Roles

### precheck

Executa validações iniciais, como conectividade Ansible, informações do sistema, IP, memória e privilege escalation.

### common

Instala pacotes base e prepara os servidores Linux.

### patch_management

Executa atualização controlada de pacotes. Essa role é separada do fluxo principal porque representa uma atividade de manutenção.

### zabbix_server

Instala e configura o Zabbix Server no Debian, incluindo MariaDB, Apache, banco de dados, schema inicial e configuração do frontend.

### zabbix_agent

Instala e configura o Zabbix Agent 2 em servidores Debian e Red Hat-like.

### zabbix_api

Usa a API do Zabbix para cadastrar hosts automaticamente no Zabbix.

### grafana

Instala o Grafana e o plugin Zabbix para integração com o Zabbix Server.

## Playbooks

### Precheck

```bash
ansible-playbook playbooks/pre_check.yml
```

### Bootstrap dos servidores

```bash
ansible-playbook playbooks/bootstrap.yml
```

### Stack de monitoramento

```bash
ansible-playbook playbooks/monitoring.yml --ask-vault-pass
```

Esse playbook executa:

1. Zabbix Server no Debian
2. Zabbix Agent 2 nos hosts Linux
3. Grafana no Debian

### Cadastro dos hosts no Zabbix

```bash
ansible-playbook playbooks/zabbix_api.yml --ask-vault-pass
```

### Manutenção

```bash
ansible-playbook playbooks/maintenance.yml
```

### Execução completa

```bash
ansible-playbook playbooks/site.yml --ask-vault-pass
```

## Ansible Vault

As senhas utilizadas pelo Zabbix ficam protegidas com Ansible Vault no diretório:

```text
vault/
```

Para executar playbooks que usam variáveis sensíveis:

```bash
ansible-playbook playbooks/monitoring.yml --ask-vault-pass
```

## Collections necessárias

Instalar dependências:

```bash
ansible-galaxy collection install -r requirements.yml
```

Collections usadas:

```yaml
collections:
  - name: ansible.posix
  - name: community.mysql
```

## Validações úteis

Verificar serviços no servidor Debian:

```bash
ansible debian -m shell -a "systemctl is-active mariadb zabbix-server apache2 zabbix-agent2 grafana-server" -b
```

Verificar frontend do Zabbix:

```bash
curl -I http://172.16.191.144/zabbix/
```

Verificar Grafana:

```bash
curl -I http://172.16.191.144:3000
```

## Status atual

* Zabbix Server instalado e funcional
* Zabbix Agent 2 instalado nos hosts Linux
* Hosts cadastrados no Zabbix via API
* Grafana instalado
* Plugin Zabbix para Grafana instalado
* Roles refatoradas e funcionando
* Execução idempotente validada com `changed=0`
* Uso de Ansible Vault para variáveis sensíveis
