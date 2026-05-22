<img width="1000" height="333" alt="image" src="https://github.com/user-attachments/assets/bb03c681-2827-45c0-82ff-bde58929c4f0" />




# Nmap-Splunk-CyberSecurity_Lab

Laboratório Prático: Auditoria de Ativos com Nmap & Centralização de Logs no Splunk Enterprise

Sobre o Projeto:

Este repositório documenta a criação de um ambiente de laboratório focado em (Reconhecimento de Rede (Footprinting)) e **Monitoramento/Análise de Segurança (SIEM)**. O objetivo prático foi escanear uma rede local para mapear ativos, identificar serviços expostos e centralizar essas informações no Splunk para visualização analítica.

 Arquitetura e Ferramentas do Ambiente:
 
* **Sistema Operacional de Ataque/Auditoria:** Kali Linux
* **Alvo dos Testes:** Roteador Doméstico e Dispositivos IoT locais (Rede Isolada)
* **Ferramenta de Varredura:** Nmap (Network Mapper) v7.98
* **Plataforma de SIEM/Logs:** Splunk Enterprise v10.2.3 (Instalado nativamente via pacote Debian)


🗺️ Roteiro de Execução Passo a Passo:

Fase 1: Identificação da Interface e Sub-rede Real
Antes de iniciar a varredura, foi necessário identificar o escopo correto da sub-rede ativa utilizando o comando: ip a

Fase 2: Descoberta de Hosts e Varredura Rápida
Utilizando privilégios de superusuário (sudo) para otimizar o envio de pacotes brutos (raw packets) e o modo rápido (-F), foi realizado o mapeamento das 100 portas mais comuns da rede:
 

sudo nmap -F 192.168.15.0/24


Descoberta: Foram identificados 4 hosts ativos, com destaque para o gateway principal (192.168.15.1) expondo as portas 22 (SSH) e 80 (HTTP).



Fase 3: Enumeração Avançada e Análise de Vulnerabilidades (NSE)
Para aprofundar a auditoria no gateway, foi utilizado o motor de scripts do Nmap (NSE) focado na categoria de vulnerabilidades catalogadas (vuln):
sudo nmap -sV --script=vuln -p 22,80 192.168.15.1


Análise Técnica:

O serviço de SSH foi identificado como Dropbear sshd 2019.78.
A porta 80 apresentou cabeçalhos de segurança modernos instalados (X-Frame-Options, X-XSS-Protection).
Os scripts do NSE confirmaram a ausência de falhas críticas comuns de aplicações web (como CSRF e XSS baseados em DOM).

Fase 4: Exportação de Logs e Integração com o SIEM (Splunk)
Para monitoramento e inteligência de dados, a varredura foi exportada no formato Grepable (-oG):
sudo nmap -sV -F 192.168.15.0/24 -oG /tmp/nmap_scan.log

O arquivo foi configurado no Splunk Enterprise como uma fonte de dados contínua (Data Monitor).

📊 Dashboards e Resultados Visuais (Splunk)

Utilizando a linguagem de busca do Splunk (SPL), foi estruturada uma query para filtrar os hosts que apresentavam portas com estado aberto:

source="/tmp/nmap_scan.log" "aberto" | stats count by host

Com os dados indexados, transformou-se a busca em uma visualização gráfica (Gráfico de Pizza), facilitando a identificação imediata dos alvos com maior superfície de exposição na rede.

<img width="1914" height="731" alt="image" src="https://github.com/user-attachments/assets/1f037bc6-983a-4a42-9634-480b6b3106f1" />





Lições Aprendidas e Mitigação (Visão de Blue Team)

Evasão de Firewall: Varreduras de Ping simples (-sn) podem falhar caso os ativos da rede bloqueiem requisições ICMP por padrão. O uso de flags como -Pn torna-se essencial nesses cenários.

Hardening: A análise do roteador demonstrou que a segurança por parte da operadora está atualizada com cabeçalhos HTTP rígidos e restrição de métodos de autenticação em nível de transporte SSH, reduzindo drasticamente riscos de força bruta locais.






