# 🛡️ Enterprise Home SOC Lab: Implantação, Monitoramento e Resposta com Wazuh SIEM (Cloud/SaaS)

> **Documentação de Engenharia e Operações de Segurança**
> **Autor / Analista Responsável:** Candidato à vaga de Assistente de SOC 1
> **Objetivo:** Estabelecer um laboratório prático de alta fidelidade para monitorar endpoints, triar alertas, analisar logs e simular ameaças com base em padrões corporativos de mercado.

---

## 📋 Sumário Executivo

Este documento descreve o planejamento, a implantação e a operação de um **SIEM (Security Information and Event Management)** com o **Wazuh Cloud**, acessado via navegador e integrado a endpoints heterogêneos.

A arquitetura em nuvem dispensa a virtualização local do servidor gerenciador e permite concentrar o esforço nas atividades de SOC:

- **Provisionar** agentes rapidamente via PowerShell;
- **Coletar** telemetria contínua dos endpoints;
- **Triar** alertas por severidade, seguindo critérios pré-definidos;
- **Monitorar** a integridade de arquivos (FIM);
- **Mapear** eventos com o framework **MITRE ATT&CK**;
- **Registrar** ocorrências e **escalar** ao N1 quando necessário.

---

## 🏗️ 1. Arquitetura e Estrutura do Laboratório

A arquitetura foi projetada de forma ágil e moderna:

- **SIEM / Central Manager:** Wazuh Cloud (instância gerenciada, acessada diretamente pelo navegador).
- **Endpoints monitorados (agents):**
  - **Host 1 – Windows:** coleta logs de eventos do Windows, execução de scripts PowerShell e integridade de diretórios críticos (FIM). Sysmon está previsto como evolução.
  - **Host 2 – Linux (planejado):** coletará syslogs, serviços de rede e eventos de autenticação.
- **Método de implantação do agente:** instalação automatizada por script executado em terminal PowerShell com privilégios administrativos.

---

## 🚀 2. Guia de Implantação e Instalação (Deployment Guide)

### 2.1. Acesso ao Wazuh Dashboard (Cloud/Navegador)

Acesse a instância do Wazuh Manager pelo navegador. O **Wazuh Dashboard** centraliza a coleta de logs, os módulos de segurança e as políticas de monitoramento. Na primeira execução, a instância indica que não há agentes registrados e oferece o botão **Deploy new agent**.

![Figura 1 - Overview do Wazuh sem agentes registrados](assets/figura-1-overview-sem-agentes.png)
*Figura 1: Acesso ao painel administrativo do Wazuh Cloud via navegador. A instância ainda não possui agentes registrados.*

### 2.2. Implantação e Conexão de Agentes via PowerShell

Registre novos endpoints pelo assistente **Agents management > Summary > Deploy new agent**. Selecione o pacote **Windows (MSI 32/64 bits)**, informe o endereço do servidor, mantenha o grupo **Default** e copie o comando gerado. O assistente exige privilégios de administrador e PowerShell 3.0 ou superior.

![Figura 2 - Assistente Deploy new agent](assets/figura-2-deploy-new-agent.png)
*Figura 2: Assistente Deploy new agent com a geração do script de instalação do agente para Windows.*

**Execução dos comandos de instalação (PowerShell como Administrador):**

1. Baixe o instalador `.msi` do agente.
2. Instale em modo silencioso, apontando para o manager em nuvem.
3. Inicie o serviço Wazuh.

```powershell
# 1) Baixar o instalador do agente
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.8-1.msi -OutFile "$env:TEMP\wazuh-agent.msi"

# 2) Instalar apontando para o manager em nuvem
msiexec.exe /i "$env:TEMP\wazuh-agent.msi" /q WAZUH_MANAGER="<SEU_MANAGER>.cloud.wazuh.com" WAZUH_REGISTRATION_PASSWORD="<CHAVE_DE_REGISTRO>"

# 3) Iniciar o serviço
NET START Wazuh
```

> ⚠️ Substitua o hostname e a chave por placeholders ao publicar e oculte esses dados nos prints.

### 2.3. Validação da Conexão do Agente

Valide o serviço no endpoint e o status do agente no dashboard.

```powershell
Get-Service -Name Wazuh
```

- **Confirme** o status `Running` no serviço.
- **Verifique** em **Endpoints** se o agente aparece como **Active**, com SO e versão corretos.

![Figura 3 - Endpoints com agente ativo](assets/figura-3-endpoints-agente-ativo.png)
*Figura 3: Endpoint `DESKTOP-19MHES6` (ID 001, 192.168.0.111) com status Active, Windows 10 Home, agente v4.14.8, grupo `default`.*

---

## 📊 3. Monitoramento de Alertas

### 3.1. Visão Geral (Overview)

Após a conexão do agente, o **Overview** passa a exibir o resumo de agentes e a contagem de alertas das últimas 24 horas, classificados por severidade.

![Figura 4 - Overview com agente ativo e alertas](assets/figura-4-overview-agente-ativo-1.png)
*Figura 4: Overview com 1 agente ativo e alertas das últimas 24 horas (Alta: 1 | Média: 319 | Baixa: 127).*

![Figura 5 - Overview com evolução da contagem de alertas](assets/figura-5-overview-agente-ativo-2.png)
*Figura 5: Overview em nova consulta (Alta: 1 | Média: 319 | Baixa: 123). A contagem de baixa severidade variou porque a janela de 24 horas é deslizante.*

**Observações:**

- **Identifiquei** 1 agente ativo e nenhum desconectado.
- **Constatei** ausência de alertas críticos (nível 15 ou superior).
- **Observei** predominância de alertas de severidade média (319 de 443, cerca de 72%).
- **Validei** a consistência dos números: 1 + 319 + 123 = 443, o mesmo total exibido no Threat Hunting.

### 3.2. Threat Hunting

Use o módulo **Threat Hunting** como tela principal de monitoramento: ele consolida volume, severidade, autenticações, técnicas MITRE ATT&CK e agentes.

![Figura 6 - Wazuh Threat Hunting Dashboard](assets/Screenshot%202026-10-10%20at%2021-05-30%20Wazuh.png)
*Figura 6: Dashboard Threat Hunting (últimas 24 horas) com volume de alertas, severidade, falhas de autenticação, técnicas MITRE ATT&CK e agentes.*

### 3.3. Interpretação dos Painéis

| Painel | Ação do analista |
|---|---|
| **Total** (443) | Avalie o volume e identifique picos anormais |
| **Level 12 or above** (1) | Priorize alertas de alta severidade |
| **Authentication failure / success** (4 / 0) | Verifique tentativas malsucedidas e sucesso posterior |
| **Top 10 Alert level evolution** | Identifique o horário e o nível dominante do pico |
| **Top 10 MITRE ATT&CK** | Relacione os alertas a táticas e técnicas |
| **Top 5 agents** | Localize o ativo que concentra os alertas |

### 3.4. Observações da Captura

- **Identifiquei** um pico de aproximadamente 440 alertas em janela curta (próximo às 21:00), concentrado no agente `DESKTOP-19MHES6`.
- **Registrei** 1 alerta de nível 12 ou superior e 4 falhas de autenticação.
- **Mapeei** três técnicas MITRE ATT&CK: Account Manipulation, Domain Policy Modification e Account Access Removal.

> 🔎 **Hipótese a validar:** picos logo após a instalação do agente costumam decorrer da ingestão inicial de eventos e da primeira avaliação de configuração (SCA). Confirme na aba **Events**, filtrando por `rule.level >= 12` e analisando a regra que gerou cada alerta antes de classificá-lo.

---

## 🧭 4. Triagem e Classificação

### 4.1. Níveis de Severidade (referência do dashboard)

| Faixa | Severidade | Ação do assistente |
|---|---|---|
| 0–6 | Baixa | Registre e acompanhe |
| 7–11 | Média | Triar e correlacione por agente/usuário |
| 12–14 | Alta | Colete evidências e **escale ao N1** |
| 15+ | Crítica | **Escale ao N1 imediatamente** |

### 4.2. Mapeamento MITRE ATT&CK

| Técnica | Tática | ID | Eventos Windows | Verificação |
|---|---|---|---|---|
| Account Manipulation | Persistence | T1098 | 4720, 4722, 4724, 4732, 4738 | Identifique conta alterada, executor e horário |
| Domain Policy Modification | Defense Evasion | T1484 | 4739, 4719 | Identifique a política alterada e o executor |
| Account Access Removal | Impact | T1531 | 4725, 4726, 4740 | Verifique conta desabilitada, excluída ou bloqueada |

### 4.3. Fluxo de Triagem

1. **Identifique** regra, nível, agente, horário e técnica MITRE.
2. **Contextualize** o ativo e correlacione com outros alertas.
3. **Aplique** o runbook correspondente (Seção 5).
4. **Classifique** como falso positivo, benigno esperado, suspeito ou incidente.
5. **Registre** a ocorrência (Seção 6).
6. **Escale** ao N1 quando atender a um critério de escalonamento.

**Critérios de escalonamento ao N1:**

- Alerta de nível 12 ou superior sem justificativa;
- Falhas de autenticação seguidas de sucesso;
- Criação ou alteração de conta privilegiada sem origem conhecida;
- Desativação ou alteração de auditoria;
- PowerShell com indícios de ofuscação ou download;
- Alteração em arquivo crítico fora de janela de mudança;
- Dúvida na classificação (na dúvida, escale).

---

## 📘 5. Runbooks de Triagem

| ID | Cenário | Event IDs | Passos | Escale quando |
|---|---|---|---|---|
| RB-01 | Falhas de autenticação | 4625 / 4624 | Anote usuário, origem, tipo de logon e contagem; procure sucesso posterior | Muitas falhas ou sucesso após falhas |
| RB-02 | Manipulação de contas | 4720, 4722, 4724, 4725, 4726, 4732, 4740 | Identifique conta afetada e executor; confirme mudança planejada | Conta em grupo admin sem justificativa |
| RB-03 | Alteração de política | 4739, 4719 | Identifique a política e o executor | Auditoria desativada ou política enfraquecida |
| RB-04 | Integridade de arquivos (FIM) | syscheck | Anote caminho, tipo de mudança, hash e usuário; correlacione com atualização legítima | Mudança em caminho crítico sem origem |
| RB-05 | PowerShell suspeito | 4104 / 4688 | Registre comando, processo pai e usuário; procure `-enc`, `IEX`, `DownloadString` | Qualquer indício de execução maliciosa |

---

## 📝 6. Registro Padronizado de Ocorrências

Registre cada alerta triado com completude e rastreabilidade.

```
ID da ocorrência: SOC-AAAAMMDD-NNN
Data/hora de detecção (UTC-3):
Analista:
Fonte: Wazuh Cloud (Threat Hunting)
Agente / Host:
Regra (rule.id / description):
Nível de severidade:
Técnica MITRE ATT&CK (ID):
Resumo do evento:
Evidências (logs, prints, hashes):
Contexto / correlação:
Runbook aplicado:
Classificação: [Falso positivo | Benigno esperado | Suspeito | Incidente]
Ação tomada:
Escalado ao N1? [Sim/Não]  Para: ____  Horário: ____
Status: [Aberto | Em análise | Escalado | Encerrado]
Justificativa de encerramento:
```

---

## 🧪 7. Simulações Controladas (Baixa Complexidade)

Execute as simulações **somente no laboratório próprio**, com ações simples e reversíveis.

| # | Simulação | Execução | Evento esperado | Runbook |
|---|---|---|---|---|
| S1 | Falhas de logon | Tente logar 4–5 vezes com senha incorreta em conta de teste | 4625 | RB-01 |
| S2 | Criação de conta | Crie e remova uma conta de teste (`net user`) | 4720 / 4726 | RB-02 |
| S3 | Conta em grupo admin | Adicione e remova a conta de teste do grupo Administrators | 4732 | RB-02 |
| S4 | Alteração de arquivo | Crie, modifique e exclua um arquivo em diretório monitorado | syscheck | RB-04 |
| S5 | PowerShell | Execute um comando inócuo (`Get-Process`) | 4104 / 4688 | RB-05 |

Para cada simulação, **execute**, **localize** o alerta no Wazuh, **aplique** o runbook e **preencha** o registro.

---

## 🧾 8. Coleta e Preservação de Evidências

- [ ] **Registre** data/hora, analista e identificador do alerta;
- [ ] **Exporte** os eventos relevantes da aba *Events* (CSV/JSON);
- [ ] **Capture** telas com horário visível;
- [ ] **Documente** hostname, IP, SO e usuário logado;
- [ ] **Calcule** o hash SHA-256 de arquivos suspeitos (`Get-FileHash <arquivo> -Algorithm SHA256`);
- [ ] **Liste** processos e conexões ativas (`Get-Process`, `netstat -ano`);
- [ ] **Armazene** tudo em pasta identificada pelo ID da ocorrência;
- [ ] **Preserve** o artefato original e analise uma cópia;
- [ ] **Entregue** o pacote ao N1 junto com o registro.

---

## 🌐 9. Fundamentos de Redes na Análise de Logs

| Protocolo | O que observar |
|---|---|
| **TCP/IP** | IP e porta de origem/destino, direção, volume e conexões externas incomuns |
| **DNS** | Domínios consultados, nomes aleatórios ou recentes e volume anormal |
| **HTTP** | URL, método, código de resposta, User-Agent e downloads de executáveis |
| **SMTP** | Remetente, destinatário, assunto, anexos e indícios de phishing |

---

## 🗺️ 10. Próximos Passos

- [ ] **Implantar** o Host 2 (Linux) e monitorar `auth.log`/`secure`;
- [ ] **Integrar** o Sysmon ao Wazuh no Windows;
- [ ] **Configurar** o FIM em diretórios críticos;
- [ ] **Concluir** a análise do pico de alertas na aba Events e classificar o caso;
- [ ] **Gerar** relatórios pelo módulo *Generate report*;
- [ ] **Estudar** para Security+ e ISO 27001 Foundation.

---

## ⚠️ Aviso

Laboratório educacional, executado em ambiente próprio e controlado. Não versione credenciais, chaves de registro nem hostnames reais.
