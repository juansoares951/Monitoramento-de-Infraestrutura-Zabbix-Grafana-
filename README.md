# Monitoramento-de-Infraestrutura-Zabbix-Grafana

Este repositório apresenta uma stack completa de monitoramento utilizando Zabbix e Grafana.
O projeto cobre instalação, configuração, integração e visualização de métricas, feito em um ambiente real de produção.

Status do ambiente: ✅ Funcionando | 🟢 Host monitorado VERDE | 📈 Dados coletados e exibidos no Grafana

🌐 Arquitetura e Fluxo de Dados
[ Windows Server 2022 ]
Zabbix Agent (10050)
        ↑
        │ Checks passivos a cada 5 minutos
        │
[ NAT / Roteador ]
Port Forward: 10050 → 192.168.0.53
        │
[ Zabbix Server - Linux ]
Processamento + Banco de dados (History + Trends)
        │
        │ API JSON-RPC
        │
[ Grafana ]
Dashboards customizados


![host-status](https://github.com/user-attachments/assets/aba2e43e-0ab2-41f0-88d8-a9efd2c88eb7)


1️⃣ Zabbix – Instalação e Configuração
Visão Geral do Ambiente: 
- Servidor Zabbix: Linux / Debian
- Agente Zabbix: Windows Server 2022
- Comunicação via checks passivos
- Acesso interno + IP público com NAT

 IP e Papéis: 
 Linux (Zabbix Server):   IP Público: 189.4.4.178 |  IP Local (LAN): 192.168.0.53
- Serviços: Zabbix Server (10051), Zabbix Agent (10050), Frontend Web

- Windows Server 2022 (Host monitorado) :
 IP Público: 186.232.81.235
 Serviço: Zabbix Agent v6.0.43 (10050)

 Conceitos importantes:
Porta 10050: Checks passivos (Server → Agent)
Porta 10051: Comunicação interna do Zabbix Server
Check passivo: Server → Agent
Check ativo (opcional): Agent → Server

Configuração do Zabbix Server (Linux)
Verificação do serviço:
systemctl status zabbix-server

 ![status](https://github.com/user-attachments/assets/5fbac258-4eee-42a1-afc8-fc9355f09153)


Verificação da porta 10051:
sudo ss -tulnp | grep 10051

![port-10051](https://github.com/user-attachments/assets/65d11b3e-c34f-4b45-8170-98cc9258e7e2)


Teste de conectividade com o agente Windows:
zabbix_get -s 186.232.81.235 -p 10050 -k system.hostname


![get-windows](https://github.com/user-attachments/assets/97def00d-b98e-4ab2-b482-05776064cb2b)


Configuração de Rede / NAT:
- Port Forward 10050 → 192.168.0.53
- Permite acesso externo ao agente Windows

![nat-10050](https://github.com/user-attachments/assets/93c18e93-30a9-4b6f-99ee-9793dad2ccdf)


Instalação do Zabbix Agent (Windows): 
- Caminho base: C:\Zabbix\
- Binários: C:\Zabbix\bin\
- Configuração: C:\Zabbix\conf\zabbix_agentd.conf
- Serviço iniciado automaticamente

![port-10050-windows](https://github.com/user-attachments/assets/49f09c3c-b2c2-4f59-9552-bd9671c287c8)


Configuração final do agente:
Server=189.4.4.178
ServerActive=189.4.4.178
Hostname=Windows_Server2022_EVEO
ListenPort=10050
StartAgents=3


![agent-conf](https://github.com/user-attachments/assets/e369d791-2553-4592-b5ee-58bfc1d31872)


Problemas comuns resolvidos:
- Connection refused → porta errada no zabbix_get
- Check access restrictions → IP do servidor não listado
- Only one usage of each socket → múltiplos agentes na mesma porta
- Configuração no Frontend Zabbix
- Host: Windows_Server2022_EVEO, Template não estava configurado para Windows by Zabbix agent 

- Apos correções: 
- Itens: 197, Triggers: 126, Gráficos: 27
- Status: 🟢 VERDE


![host-status](https://github.com/user-attachments/assets/e146d0ba-d7d9-4eab-be09-10f8476f5322)

![triggers](https://github.com/user-attachments/assets/738efe97-5670-421e-b5af-34533d58cfb0)

![graphs1](https://github.com/user-attachments/assets/04340582-9a8f-41e2-8cd6-7e97248a6388)

Coleta de dados:
- Intervalo: 5 minutos | History: 7 dias | Trends: 90 dias
- Uso estimado: ~12 MB

2️⃣ Grafana – Instalação e Integração
Pré-requisitos :
- Servidor Debian com Zabbix Server e Frontend instalados
- Banco de dados configurado
- Hosts Windows monitorados
- Acesso root/sudo

Instalação do Grafana
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
sudo curl https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/grafana-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/grafana-archive-keyring.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install -y grafana

Configuração inicial:
- Porta HTTP: 7777
- Root path: /grafana
- Usuário: admin/admin

![2grafana ini](https://github.com/user-attachments/assets/9a08895a-539b-4a45-b76c-d6a3bfa616fc)
![3running](https://github.com/user-attachments/assets/3e76e502-e86a-4eb5-933a-7480eebd1a03)

Proxy Reverso Apache:
- Redireciona /grafana/ → http://127.0.0.1:7777/
- Permite acesso externo via IP público

![4grafana conf](https://github.com/user-attachments/assets/e0b077a6-7a3d-444a-b2bd-5098a9012aa6)

Acesso ao Grafana:
- Interno: http://192.168.0.53:7777/grafana/login
- Externo: http://189.4.4.178:7777/login

![5login](https://github.com/user-attachments/assets/50971fdf-bf61-474e-a5a5-91fb6106400b)

Plugin Zabbix: alexanderzobnin-zabbix-app

![6pluggin_zabbix](https://github.com/user-attachments/assets/aa47c5fa-bfca-447d-900d-d2a65e9cde88)

Configuração da Data Source: 
- Conexão via API do Zabbix
- Usuário read-only
- Dashboards criados manualmente: CPU, Memória, Disco, Rede, Usuarios




![8configurar_dashboard](https://github.com/user-attachments/assets/0889039c-ce2d-43f4-b3e7-652f4f9796b9)
![9dashboard_finalizado](https://github.com/user-attachments/assets/90e46094-e803-4678-a548-bec4d52af331)
![10dashboards](https://github.com/user-attachments/assets/b19f898e-a82d-4801-ae90-88685dd70b6d)


✅ Resultado Final da Stack

Host monitorado: 
- Windows_Server2022_EVEO  
- Windows_Server2022_Local
- Zabbix server
  
- Ambiente 100% funcional
- Dados coletados corretamente
- Triggers operacionais
- Dashboards do Grafana visíveis externamente
