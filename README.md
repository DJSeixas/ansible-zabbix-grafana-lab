# Ansible Zabbix Grafana Lab

Projeto prático de automação com Ansible para provisionar e configurar uma stack de monitoramento Linux com **Zabbix Server**, **Zabbix Agent 2**, **Grafana** e **Chrony/NTP**.

O objetivo do projeto é praticar automação de infraestrutura usando uma estrutura organizada com **playbooks**, **roles**, **variáveis**, **templates Jinja2**, **handlers**, **Ansible Vault** e integração com a **API do Zabbix**.

## Objetivo

Automatizar servidores Linux Debian, Rocky Linux e Oracle Linux com Ansible, incluindo:

* Configuração base dos servidores
* Instalação de pacotes comuns
* Sincronização de horário com Chrony/NTP
* Instalação do Zabbix Server no Debian
* Instalação do Zabbix Agent 2 nos hosts Linux
* Cadastro automático dos hosts no Zabbix via API
* Instalação do Grafana
* Instalação do plugin Zabbix para Grafana
* Uso de Ansible Vault para variáveis sensíveis
* Separação da automação por roles
* Execução idempotente dos playbooks

## Ambiente

| Host           | Sistema      | Função                                                    |
| -------------- | ------------ | --------------------------------------------------------- |
| `srv_debian01` | Debian       | Zabbix Server, Grafana, Zabbix Agent e NTP server interno |
| `srv_rocky01`  | Rocky Linux  | Zabbix Agent e cliente NTP                                |
| `srv_oracle01` | Oracle Linux | Zabbix Agent e cliente NTP                                |

## Estrutura principal

```text
ansible-lab/
├── ansible.cfg
├── inventory.ini
├── requirements.yml
├── README.md
├── group_vars/
│   ├── all/
│   │   └── vars.yml
│   ├── debian.yml
│   ├── redhat.yml
│   ├── rocky.yml
│   └── oracle.yml
├── host_vars/
│   └── srv_debian01.yml
├── vault/
│   └── zabbix.yml
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
    ├── time_sync/
    ├── zabbix_server/
    ├── zabbix_agent/
    ├── zabbix_api/
    └── grafana/
```

## Roles

### `precheck`

Executa validações iniciais nos servidores, como:

* Conectividade Ansible
* Informações do sistema operacional
* Endereço IPv4
* Memória disponível
* Validação de privilege escalation com `become`

### `common`

Instala pacotes base e prepara os servidores Linux para as próximas etapas da automação.

### `patch_management`

Executa atualização controlada de pacotes.

Essa role é separada do fluxo principal porque representa uma atividade de manutenção e não deve ser executada automaticamente em todo deploy.

### `time_sync`

Instala e configura Chrony/NTP nos servidores Linux.

No desenho atual do laboratório:

* `srv_debian01` atua como servidor NTP interno
* `srv_rocky01` e `srv_oracle01` sincronizam horário com o Debian
* O Debian sincroniza com servidores NTP externos
* A configuração é gerada via template Jinja2
* O serviço é habilitado e iniciado automaticamente

### `zabbix_server`

Instala e configura o Zabbix Server no Debian, incluindo:

* Repositório oficial do Zabbix
* MariaDB
* Apache
* Zabbix Server
* Zabbix frontend
* Criação do banco de dados
* Criação do usuário do banco
* Importação do schema inicial
* Configuração do `zabbix_server.conf`
* Configuração do frontend Apache/PHP

### `zabbix_agent`

Instala e configura o Zabbix Agent 2 em servidores Debian e Red Hat-like.

A role utiliza template Jinja2 para gerar a configuração do Agent e handlers para reiniciar o serviço quando necessário.

### `zabbix_api`

Usa a API JSON-RPC do Zabbix para cadastrar automaticamente os hosts Linux no Zabbix.

Fluxo principal:

1. Login na API do Zabbix
2. Busca do host group
3. Busca do template
4. Verificação de hosts existentes
5. Criação apenas dos hosts ausentes
6. Validação de erros retornados pela API

### `grafana`

Instala o Grafana no Debian e adiciona o plugin Zabbix para integração com o Zabbix Server.

## Playbooks

### `pre_check.yml`

Executa validações iniciais nos servidores Linux.

```bash
ansible-playbook playbooks/pre_check.yml
```

### `bootstrap.yml`

Executa a preparação base dos servidores.

Inclui:

1. Role `common`
2. Role `time_sync`

```bash
ansible-playbook playbooks/bootstrap.yml
```

### `monitoring.yml`

Instala e configura a stack de monitoramento.

Inclui:

1. Zabbix Server no Debian
2. Zabbix Agent 2 nos hosts Linux
3. Grafana no Debian

```bash
ansible-playbook playbooks/monitoring.yml --ask-vault-pass
```

### `zabbix_api.yml`

Cadastra os hosts Linux no Zabbix via API.

```bash
ansible-playbook playbooks/zabbix_api.yml --ask-vault-pass
```

### `maintenance.yml`

Executa atualização controlada de pacotes.

```bash
ansible-playbook playbooks/maintenance.yml
```

### `site.yml`

Executa o fluxo principal do projeto.

```bash
ansible-playbook playbooks/site.yml --ask-vault-pass
```

## Ansible Vault

As variáveis sensíveis utilizadas pelo Zabbix ficam protegidas com Ansible Vault no diretório:

```text
vault/
```

Exemplo de execução com Vault:

```bash
ansible-playbook playbooks/monitoring.yml --ask-vault-pass
```

O arquivo de Vault deve permanecer criptografado.

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

## Status atual

* Estrutura refatorada em roles
* Zabbix Server instalado e funcional
* Zabbix Agent 2 instalado nos hosts Linux
* Hosts cadastrados no Zabbix via API
* Grafana instalado
* Plugin Zabbix para Grafana instalado
* Chrony/NTP configurado nos servidores Linux
* Debian atuando como servidor NTP interno do lab
* Rocky Linux e Oracle Linux sincronizando com o Debian
* Uso de Ansible Vault para variáveis sensíveis
* Execução idempotente validada com `changed=0`


## Observação

Este projeto é um laboratório de estudo e portfólio técnico. O foco é demonstrar evolução prática em automação, monitoramento, administração Linux e operação de infraestrutura usando ferramentas comuns em ambientes corporativos.
