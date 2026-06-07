Objetivo:
Automatizar servidores Linux Debian, Rocky e Oracle com Ansible.

Componentes:
- Pacotes base
- Zabbix Server no Debian
- Zabbix Agent 2 nos hosts Linux
- Cadastro de hosts no Zabbix via API
- Preparação futura para Grafana

Hosts:
- srv_debian01
- srv_rocky01
- srv_oracle01

Fluxo:
1. preCheck.yml
2. basePackages.yml
3. installZabbixServer.yml
4. installZabbixAgent.yml
5. registerZabbixHosts.yml
6. installGrafana.yml
