# 🛡️ Enterprise Home SOC Lab: Implantação, Monitoramento e Resposta com Wazuh SIEM (Cloud/SaaS)

> **Documentação de Engenharia e Operações de Segurança**  
> **Autor / Analista Responsável:** Candidato à vaga de Assistente de SOC 1  
> **Objetivo:** Estabelecer um laboratório prático de alta fidelidade para monitoramento de endpoints, triagem de alertas, análise de logs e simulação de ameaças com base em padrões corporativos de mercado.

---

## 📋 Sumário Executivo
Este documento detalha o planejamento, implantação e operação de um Security Information and Event Management (SIEM) utilizando o **Wazuh Cloud** acessado via navegador, integrado a endpoints heterogêneos. A arquitetura em nuvem elimina o overhead de virtualização local para o servidor gerenciador, focando no provisionamento rápido de agentes via **PowerShell**, coleta contínua de telemetria, triagem baseada em severidade, monitoramento de integridade de arquivos (FIM) e mapeamento com o framework **MITRE ATT&CK**.

---

## 🏗️ 1. Arquitetura e Estrutura do Laboratório

A arquitetura do laboratório foi projetada de forma ágil e moderna:

*   **SIEM / Central Manager:** Wazuh Cloud (Instância gerenciada acessada diretamente via navegador web)
*   **Endpoints Monitorados (Agents):**
    *   **Host 1:** Windows (Monitoramento de logs de eventos do Windows, execução de scripts PowerShell,Sysmon e integridade de diretórios críticos)
    *   **Host 2:** Linux (Monitoramento de syslogs, serviços de rede e autenticação)
*   **Método de Implantação do Agente:** Instalação automatizada via script executado em terminal PowerShell com privilégios administrativos.

---

## 🚀 2. Guia de Implantação e Instalação (Deployment Guide)

### 2.1. Acesso ao Wazuh Dashboard (Cloud/Navegador)
A instância do Wazuh Manager foi provisionada e acessada via navegador web. A interface unificada do `Wazuh Dashboard` gerencia centralidamente a coleta de logs e as políticas de segurança:

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2020-16-04%20Wazuh.png" width="85%" alt="Acesso ao Console do Wazuh Cloud">
  <p><em>Figura 1: Acesso ao painel administrativo do Wazuh Cloud via navegador web.</em></p>
</div>

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2020-20-40%20Wazuh.png" width="85%" alt="Wazuh Dashboard Initial View">
  <p><em>Figura 2: Visão geral do Wazuh Dashboard e consolidação dos módulos de monitoramento.</em></p>
</div>

### 2.2. Implantação e Conexão de Agentes via PowerShell
O registro de novos endpoints foi realizado através do assistente de configuração (**Agents management > Summary > Deploy new agent**). O assistente fornece as chaves de registro criptográficas e o comando de download do pacote do agente:

<div align="center">
  <img src="./assets/Screenshot%202026-10-10%20at%2020-47-27%20Wazuh.png" width="85%" alt="Assistente de Deploy do Agente Wazuh">
  <p><em>Figura 3: Geração de chave e script de instalação do agente para o ambiente Windows no assistente do Wazuh.</em></p>
</div>

#### Execução dos Comandos de Instalação (PowerShell Executado como Administrador):

Para realizar o download do instalador `.msi`, configurar o endereço do servidor gerenciador em nuvem e iniciar o serviço, executou-se a seguinte sequência no terminal PowerShell do endpoint:

```powershell
Invoke-WebRequest -Uri [https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.8-1.msi](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.8-1.msi) -OutFile $env:temp\wazuh-agent; msiexec.exe /i$env:temp\wazuh-agent /q WAZUH_MANAGER="y9pbiety148j.cloud.wazuh.com" WAZUH_REGISTRATION_PASSWORD="<CHAVE_DE_REGISTRO>"
NET START Wazuh
