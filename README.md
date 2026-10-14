# 🛡️ Enterprise Home SOC Lab: Implantação, Monitoramento e Resposta com Wazuh SIEM

> **Documentação de Engenharia e Operações de Segurança**  
> **Autor / Analista Responsável:** Candidato à vaga de Assistente de SOC 1  
> **Objetivo:** Estabelecer um laboratório prático de alta fidelidade para monitoramento de endpoints, triagem de alertas, análise de logs e simulação de ameaças com base em padrões corporativos de mercado.

---

## 📋 Sumário Executivo
Este documento detalha o planejamento, dimensionamento, implantação e operação de um Security Information and Event Management (SIEM) utilizando o **Wazuh**, integrado a ambientes heterogêneos (Windows e Linux). O laboratório foi construído para simular cenários reais de incidentes de segurança, permitindo o desenvolvimento de competências essenciais para a operação de um SOC de Nível 1: monitoramento contínuo de infraestrutura, triagem baseada em severidade, análise de integridade de arquivos (FIM), detecção de ameaças e mapeamento com o framework **MITRE ATT&CK**.

---

## 🏗️ 1. Arquitetura e Dimensionamento de Infraestrutura

Para garantir a estabilidade e a viabilidade do laboratório em ambiente local sem comprometer a performance, o dimensionamento de hardware seguiu as diretrizes recomendadas pela documentação oficial do Wazuh para ambientes de teste e validação:

*   **Hipervisor:** Oracle VirtualBox (Ambiente virtualizado isolado)
*   **Wazuh Server / Indexer / Dashboard (All-in-One OVA):**
    *   *Recursos Alocados:* 4 vCPUs, 8 GB de RAM e 50 GB de Armazenamento (dimensionado para gerenciar até 25 agentes simultâneos).
*   **Endpoints Monitorados (Agents):**
    *   **Host 1:** Windows 10 / Windows Server (Monitoramento de logs de eventos do Windows, PowerShell e integridade de diretórios críticos).
    *   **Host 2:** Ubuntu Linux (Monitoramento de syslogs, serviços de rede e pacotes instalados).

---

## 🚀 2. Guia de Implantação e Instalação (Deployment Guide)

### 2.1. Instalação e Inicialização do Wazuh Manager (SIEM)
A implantação do servidor central foi realizada utilizando a imagem oficial OVA no VirtualBox, permitindo o provisionamento rápido do ecossistema completo (Wazuh Indexer, Server e Dashboard).

Após a inicialização da máquina virtual, o acesso ao painel de controle (`Wazuh Dashboard`) é disponibilizado via navegador web através do IP atribuído à interface de rede:

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2020-16-04%20Wazuh.png" width="85%" alt="Configuração da VM no VirtualBox">
  <p><em>Figura 1: Ajuste de recursos de hardware e configuração de rede para o servidor Wazuh.</em></p>
</div>

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2020-20-40%20Wazuh.png" width="85%" alt="Wazuh Dashboard Initial View">
  <p><em>Figura 2: Visão inicial do Wazuh Dashboard após a inicialização bem-sucedida do servidor central e consolidação dos módulos de monitoramento.</em></p>
</div>

### 2.2. Gerenciamento e Instalação de Agentes nos Endpoints
A conexão dos hosts ao servidor central é gerenciada diretamente pelo assistente de configuração integrado na interface web (**Agents management > Summary > Deploy new agent**), onde são gerados os pacotes e comandos específicos para cada sistema operacional:

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2020-47-27%20Wazuh.png" width="85%" alt="Wazuh Deploy Agent Wizard">
  <p><em>Figura 3: Assistente de implantação de novos agentes no Wazuh Dashboard, exibindo os parâmetros de conexão segura via chave criptográfica.</em></p>
</div>

#### Comandos de Instalação Automatizada:

*   **Ambiente Windows (PowerShell Executado como Administrador):**
    ```powershell
    Invoke-WebRequest -Uri [https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.8-1.msi](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.8-1.msi) -OutFile $env:temp\wazuh-agent; msiexec.exe /i$env:temp\wazuh-agent /q WAZUH_MANAGER="<IP_DO_SERVIDOR_WAZUH>" WAZUH_REGISTRATION_PASSWORD="<CHAVE_DE_REGISTRO>"
    NET START Wazuh
    ```

*   **Ambiente Linux (Ubuntu / Debian):**
    ```bash
    curl -s [https://packages.wazuh.com/key/GPG-KEY-WAZUH](https://packages.wazuh.com/key/GPG-KEY-WAZUH) | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
    echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] [https://packages.wazuh.com/4.x/apt/](https://packages.wazuh.com/4.x/apt/) stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
    sudo apt update && sudo apt install wazuh-agent
    sudo WAZUH_MANAGER="<IP_DO_SERVIDOR_WAZUH>" systemctl enable wazuh-agent
    sudo systemctl start wazuh-agent
    ```

Após a execução dos comandos, a telemetria do endpoint é estabelecida de forma segura, registrando o host ativo no painel central:

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2021-04-23%20Wazuh.png" width="85%" alt="Active Agents Dashboard">
  <p><em>Figura 4: Status dos agentes ativos conectados ao Wazuh Manager, validando a integridade da coleta de dados em tempo real.</em></p>
</div>

---

## 🔍 3. Monitoramento, Engenharia de Alertas e Níveis de Severidade

O Wazuh categoriza os eventos de segurança em uma escala de criticidade que varia de **0 a 15**. Alertas de nível intermediário a alto (ex: nível $\ge 3$) disparam notificações automatizadas para os fluxos de operação e canais de comunicação.

A consolidação da postura de segurança dos endpoints monitorados é visualizada através dos módulos integrados de *Endpoint Security*, detecção de vulnerabilidades e auditoria de configuração:

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2021-05-30%20Wazuh.png" width="85%" alt="Security Modules Overview">
  <p><em>Figura 5: Painel consolidado de módulos de segurança (Security Operations, Vulnerability Detection e Integrity Monitoring).</em></p>
</div>

---

## 🎯 4. Threat Hunting e Mapeamento MITRE ATT&CK

Para cumprir com os requisitos de análise de ameaças de um analista SOC N1, o laboratório foi submetido a cenários de validação de detecção baseados no framework **MITRE ATT&CK**:

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2021-16-04%20Wazuh.png" width="85%" alt="MITRE ATT&CK and Threat Hunting Panel">
  <p><em>Figura 6: Painel analítico de Threat Hunting correlacionando falhas de autenticação, anomalias baseadas em host e táticas mapeadas pelo MITRE ATT&CK.</em></p>
</div>

---

## 🛠️ 5. Competências Demonstradas no Laboratório
*   **Fundamentos de Redes e Protocolos:** Análise de tráfego TCP/IP, DNS, HTTP e SMTP correlacionados aos logs de segurança.
*   **Sistemas Operacionais:** Administração, endurecimento e monitoramento de logs em ambientes Windows e Linux.
*   **Operação de SIEM:** Implantação, gestão de agentes, configuração de regras de alerta e triagem de falsos positivos utilizando o Wazuh.
*   **Disciplina Operacional e Rastreabilidade:** Documentação padronizada de incidentes de baixa complexidade, alinhada às exigências de plantões de SOC (24x7).

---
*Repositório mantido para fins de demonstração técnica em processos seletivos de Segurança Defensiva e Operações de SOC.*
