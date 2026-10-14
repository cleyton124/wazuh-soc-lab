# 🛡️ Home SOC Lab: Monitoramento, Triagem e Resposta com Wazuh SIEM (Cloud/SaaS)

> **Documentação de Operações de Segurança (SOC N0/Assistente)**
> **Autor:** Candidato à vaga de Assistente de SOC 1 (CSIRT)
> **Objetivo:** Laboratório prático que reproduz a rotina de um SOC: monitorar alertas seguindo runbooks, triar eventos por critérios pré-definidos, registrar ocorrências de forma padronizada, coletar evidências e escalar para o N1 quando necessário.

---

## 📋 Sumário Executivo

Este laboratório utiliza o **Wazuh Cloud** (SIEM/XDR gerenciado, acessado pelo navegador) integrado a endpoints reais. A arquitetura em nuvem elimina a necessidade de hospedar o servidor gerenciador localmente e permite concentrar o esforço no que importa para a função de analista de SOC: **telemetria, triagem, registro e escalonamento**.

O foco não é "customizar o SIEM", mas **operá-lo como um assistente de SOC**: ler o dashboard, aplicar critérios de severidade, seguir runbooks, documentar com rastreabilidade e saber quando escalar.

## 🎯 Alinhamento com a Vaga (Assistente de SOC 1 – CSIRT)

| Requisito / Responsabilidade da vaga | Como o laboratório demonstra |
|---|---|
| Monitorar alertas seguindo runbooks e checklists | Seção 5: fluxo de triagem e runbooks por tipo de alerta |
| Triar eventos com critérios pré-estabelecidos e escalar ao N1 | Seção 5.2 (matriz de severidade) e 5.3 (critérios de escalonamento) |
| Registrar ocorrências com padronização, completude e rastreabilidade | Seção 6: template de registro de ocorrência + caso de exemplo |
| Uso operacional de SIEM / EDR | Wazuh Dashboard (Threat Hunting, MITRE ATT&CK, FIM, Agents) |
| Atuação em incidentes de baixa complexidade | Seção 7: simulações controladas (falhas de autenticação, conta, FIM) |
| Apoio em investigações e coleta de evidências | Seção 8: checklist de coleta e preservação de evidências |
| Redes (TCP/IP, DNS, HTTP, SMTP) | Seção 9: campos de rede a observar na análise de logs |
| Windows e Linux | Endpoint Windows ativo; endpoint Linux em roadmap (Seção 10) |
| MITRE ATT&CK | Seção 4.2: mapeamento dos alertas observados em táticas e técnicas |
| Comunicação técnica escrita | Toda a documentação em formato de runbook e relatório |

---

## 🏗️ 1. Arquitetura e Estrutura do Laboratório

| Componente | Função | Status |
|---|---|---|
| **Wazuh Cloud** (Manager + Indexer + Dashboard) | SIEM central, acessado via navegador | ✅ Ativo |
| **Host 1 – Windows 10** (`DESKTOP-19MHES6`) | Logs de eventos do Windows, PowerShell, integridade de diretórios (FIM) | ✅ Agente ativo |
| **Host 2 – Linux** | syslog, autenticação (`auth.log`/`secure`), serviços de rede | 🔜 Planejado (Seção 10) |

**Método de implantação:** instalação via PowerShell com privilégios administrativos, usando o comando gerado pelo assistente *Deploy new agent*.

---

## 🚀 2. Guia de Implantação (Deployment Guide)

### 2.1. Acesso ao Wazuh Dashboard

A instância do Wazuh foi provisionada em nuvem e acessada pelo navegador. A interface unificada centraliza coleta de logs, módulos de segurança e políticas.

<!-- Mantenha aqui as Figuras 1 e 2 já existentes na pasta assets/ -->
*Figura 1: Acesso ao painel administrativo do Wazuh Cloud via navegador.*

*Figura 2: Visão geral do Wazuh Dashboard e módulos de monitoramento.*

### 2.2. Registro de endpoints via PowerShell

O registro foi feito em **Agents management > Summary > Deploy new agent**. O assistente fornece o pacote, o endereço do servidor e a chave de registro.

<!-- Mantenha aqui a Figura 3 já existente na pasta assets/ -->
*Figura 3: Geração do comando de instalação do agente para Windows.*

**Comandos executados (PowerShell como Administrador):**

```powershell
# 1) Baixar o instalador do agente
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.8-1.msi -OutFile "$env:TEMP\wazuh-agent.msi"

# 2) Instalar em modo silencioso apontando para o manager em nuvem
msiexec.exe /i "$env:TEMP\wazuh-agent.msi" /q WAZUH_MANAGER="<SEU_MANAGER>.cloud.wazuh.com" WAZUH_REGISTRATION_PASSWORD="<CHAVE_DE_REGISTRO>"

# 3) Iniciar o serviço
NET START Wazuh
```

> ⚠️ **Boas práticas de segurança:** nunca publique a chave de registro nem o hostname real do manager em repositórios públicos. Use placeholders no texto e **oculte/borre** esses dados nos prints.

### 2.3. Validação da instalação

```powershell
Get-Service -Name Wazuh                      # Status esperado: Running
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 20   # Conexão com o manager
```

No dashboard, confirmar o agente em **Endpoints** com status **Active**, sistema operacional correto e versão do agente.

<!-- Mantenha aqui o print de Endpoints (agente Active) já existente -->
*Figura 4: Endpoint `DESKTOP-19MHES6` com status Active (Windows 10, Wazuh v4.14.8).*

---

## 📊 3. Operação: Visão de Threat Hunting

Após o agente conectar, a telemetria passa a alimentar o módulo **Threat Hunting**, principal tela de trabalho do analista de monitoramento.

![Wazuh Threat Hunting Dashboard](assets/Screenshot%202026-10-10%20at%2021-05-30%20Wazuh.png)
*Figura 5: Dashboard Threat Hunting (últimas 24 horas) com indicadores de volume, severidade, falhas de autenticação, técnicas MITRE ATT&CK e agentes.*

### 3.1. Leitura do dashboard (o que cada painel responde)

| Painel | Pergunta que o analista responde |
|---|---|
| **Total** (443) | Qual o volume de alertas no período? Há pico anormal? |
| **Level 12 or above alerts** (1) | Existe algo de alta severidade que exige atenção imediata? |
| **Authentication failure** (4) / **success** (0) | Há tentativas de acesso malsucedidas? Houve sucesso depois (possível brute force bem-sucedido)? |
| **Top 10 Alert level evolution** | Quando o volume subiu e quais níveis dominaram? |
| **Top 10 MITRE ATT&CK** | Quais táticas/técnicas estão associadas aos alertas? |
| **Top 5 agents** | Qual ativo concentra os alertas? |

### 3.2. Observações da captura

- **Pico concentrado** de ~440 alertas em uma janela curta, próximo ao fim do período (~21:00), concentrado no agente `DESKTOP-19MHES6`.
- **1 alerta nível 12 ou superior** e **4 falhas de autenticação** no período.
- **Técnicas MITRE listadas:** Account Manipulation, Domain Policy Modification, Account Access Removal.

> 🔎 **Hipótese a validar (não conclusão):** picos logo após a instalação do agente costumam estar ligados à ingestão inicial de eventos e à primeira avaliação de configuração (SCA). A hipótese deve ser confirmada abrindo a aba **Events**, filtrando por `rule.level >= 12` e por `rule.groups`, e verificando qual regra gerou cada alerta antes de classificá-lo como falso positivo ou atividade real.

---

## 🧭 4. Fundamentos de Triagem

### 4.1. Níveis de severidade do Wazuh (referência do próprio dashboard)

| Faixa | Severidade | Postura do assistente |
|---|---|---|
| 0–6 | Baixa | Registrar e acompanhar, seguir runbook |
| 7–11 | Média | Triar, correlacionar com outros eventos do mesmo agente |
| 12–14 | Alta | Triar imediatamente, coletar evidências, **escalar ao N1** |
| 15+ | Crítica | **Escalar ao N1 imediatamente** |

### 4.2. Mapeamento MITRE ATT&CK dos alertas observados

| Técnica observada | Tática | ID | Eventos Windows típicos | O que verificar |
|---|---|---|---|---|
| Account Manipulation | Persistence | T1098 | 4720, 4722, 4724, 4738, 4732 | Quem alterou, qual conta, horário, se foi ação administrativa esperada |
| Domain Policy Modification | Defense Evasion / Privilege Escalation | T1484 | 4739, 4719 | Mudança de política de auditoria/senha/bloqueio, usuário executor |
| Account Access Removal | Impact | T1531 | 4725, 4726, 4740 | Conta desabilitada/excluída/bloqueada, impacto em usuários legítimos |

> Em ambiente doméstico, alertas dessas técnicas frequentemente decorrem de ações legítimas do próprio operador (ex.: criar usuário de teste). A triagem consiste em **correlacionar com a atividade planejada** antes de classificar.

---

## 🧪 5. Runbooks de Triagem (Operação sob Supervisão)

### 5.1. Fluxo geral de triagem

```
Alerta recebido
   │
   ├─► 1. Identificar: regra (rule.id), nível, agente, horário, MITRE
   ├─► 2. Contextualizar: o ativo é crítico? há outros alertas do mesmo agente/usuário/IP?
   ├─► 3. Aplicar o runbook correspondente (5.4)
   ├─► 4. Classificar: falso positivo | evento benigno esperado | suspeito | incidente
   ├─► 5. Registrar ocorrência (Seção 6)
   └─► 6. Atendeu critério de escalonamento? ── Sim ──► Escalar ao N1 com evidências
                                              └─ Não ──► Encerrar com justificativa
```

> O assistente **segue o runbook e escala**. Decisões de contenção ou mudança de configuração são do N1 ou superior.

### 5.2. Critérios de escalonamento ao N1

Escalar quando houver **qualquer um** dos itens:

- Alerta nível **≥ 12** não explicado por atividade planejada.
- Falhas de autenticação repetidas **seguidas de sucesso** na mesma conta ou origem.
- Criação/alteração de conta privilegiada ou de grupo administrativo sem origem conhecida.
- Alteração de políticas de auditoria ou desativação de logging.
- Execução suspeita de PowerShell (comandos codificados, download e execução).
- Alteração em arquivos críticos monitorados por FIM fora de janela de mudança.
- Dúvida sobre classificação (na dúvida, **escalar**).

### 5.3. Runbooks por tipo de alerta

#### RB-01: Falhas de autenticação (Windows, Event ID 4625)

| Etapa | Ação |
|---|---|
| 1 | Anotar usuário alvo, IP/host de origem, tipo de logon (`logonType`), contagem e janela de tempo |
| 2 | Verificar se há **4624** (sucesso) logo após as falhas, para a mesma conta/origem |
| 3 | Verificar se a origem é interna/conhecida; consultar reputação do IP externo |
| 4 | Poucas falhas, origem conhecida, sem sucesso → benigno (erro de digitação). Muitas falhas ou sucesso posterior → **escalar** |
| 5 | Registrar ocorrência |

#### RB-02: Manipulação/remoção de contas (Event IDs 4720, 4722, 4724, 4725, 4726, 4732, 4738, 4740)

| Etapa | Ação |
|---|---|
| 1 | Identificar conta afetada, **conta executora** e horário |
| 2 | Confirmar se corresponde a mudança planejada/chamado |
| 3 | Verificar se a conta foi adicionada a grupo privilegiado (Administrators) |
| 4 | Sem justificativa → **escalar** com evidências |

#### RB-03: Alteração de política (Event IDs 4739, 4719)

| Etapa | Ação |
|---|---|
| 1 | Identificar qual política foi alterada (auditoria, senha, bloqueio) |
| 2 | Identificar o usuário/processo executor |
| 3 | Desativação de auditoria ou enfraquecimento de política sem justificativa → **escalar** |

#### RB-04: Integridade de arquivos (FIM / syscheck)

| Etapa | Ação |
|---|---|
| 1 | Anotar caminho, tipo de mudança (added/modified/deleted), hash antes/depois, usuário |
| 2 | Verificar se o caminho é crítico (sistema, inicialização, binários) |
| 3 | Correlacionar com instalação/atualização legítima |
| 4 | Mudança em caminho crítico sem origem conhecida → **escalar** |

#### RB-05: PowerShell suspeito (Event ID 4104 / 4688)

| Etapa | Ação |
|---|---|
| 1 | Registrar linha de comando, processo pai, usuário e horário |
| 2 | Verificar indícios: `-enc`, `IEX`, `DownloadString`, `-ExecutionPolicy Bypass`, conexão externa |
| 3 | Qualquer indício → **escalar** e preservar o conteúdo do script |

---

## 📝 6. Registro Padronizado de Ocorrências

Todo alerta triado deve gerar um registro **completo e rastreável**.

### 6.1. Template

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
Evidências (campos do log, prints, hashes): 
Contexto/correlação: 
Runbook aplicado: RB-0X
Classificação: [Falso positivo | Benigno esperado | Suspeito | Incidente]
Ação tomada: 
Escalado ao N1? [Sim/Não]  Para: ____  Horário: ____
Status: [Aberto | Em análise | Escalado | Encerrado]
Justificativa de encerramento: 
```

### 6.2. Exemplo preenchido a partir da Figura 5

> Os campos marcados como **(a confirmar)** dependem da inspeção da aba *Events* e não devem ser preenchidos por suposição.

```
ID da ocorrência: SOC-20261010-001
Data/hora de detecção: 10/10/2026 ~21:00 (UTC-3)
Analista: <seu nome>
Fonte: Wazuh Cloud (Threat Hunting)
Agente / Host: DESKTOP-19MHES6 (192.168.0.111)
Regra (rule.id / description): (a confirmar)
Nível de severidade: 12+ (1 alerta)
Técnica MITRE ATT&CK: T1098 / T1484 / T1531 (conforme painel)
Resumo do evento: Pico de ~440 alertas em janela curta no agente Windows,
  com 1 alerta nível ≥12, 4 falhas de autenticação e técnicas de
  manipulação de conta e política.
Evidências: Print do dashboard (Figura 5); export dos eventos filtrados (a anexar)
Contexto/correlação: Ocorrido após instalação do agente e testes no lab (a confirmar)
Runbook aplicado: RB-01, RB-02, RB-03
Classificação: (a definir após análise dos eventos)
Escalado ao N1? Conforme critério 5.2
Status: Em análise
```

---

## 🔍 7. Simulações Controladas (Incidentes de Baixa Complexidade)

Todas as simulações são feitas **apenas no próprio laboratório**, com ações simples e reversíveis, para gerar alertas reais e praticar a triagem.

| # | Simulação | Como executar (lab) | Evento esperado | Runbook |
|---|---|---|---|---|
| S1 | Falhas de logon | Tentar logar 4–5 vezes com senha incorreta em conta de teste | 4625 | RB-01 |
| S2 | Criação de conta | `net user labtest <senha> /add` e remoção em seguida | 4720 / 4726 | RB-02 |
| S3 | Conta em grupo admin | Adicionar e remover a conta de teste do grupo local Administrators | 4732 | RB-02 |
| S4 | Alteração de arquivo (FIM) | Criar/modificar/excluir arquivo em diretório monitorado | syscheck | RB-04 |
| S5 | PowerShell | Executar comando inócuo (`Get-Process`) e observar o log de script block | 4104 / 4688 | RB-05 |
| S6 | Arquivo de teste antivírus | Baixar o arquivo **EICAR** (inofensivo, padrão de teste) | Detecção do AV/Defender | RB-04 |

Para cada simulação: executar → localizar o alerta no Wazuh → aplicar o runbook → preencher o registro (Seção 6).

---

## 🧾 8. Apoio em Investigações: Coleta de Evidências

**Checklist de coleta (estado mínimo preservado antes de qualquer ação):**

- [ ] Registrar data/hora, analista e fonte (identificador do alerta)
- [ ] Exportar os eventos relevantes do Wazuh (aba *Events*, formato CSV/JSON)
- [ ] Capturar telas do alerta com horário visível
- [ ] Registrar dados do host: hostname, IP, SO, usuário logado
- [ ] Coletar hash (SHA-256) de arquivos suspeitos: `Get-FileHash <arquivo> -Algorithm SHA256`
- [ ] Anotar processos e conexões ativas no momento: `Get-Process`, `netstat -ano`
- [ ] Armazenar evidências em pasta identificada pelo ID da ocorrência
- [ ] Não alterar nem apagar o artefato original; trabalhar sobre cópia
- [ ] Entregar o pacote ao N1 junto com o registro da ocorrência

---

## 🌐 9. Fundamentos de Redes na Análise de Logs

| Protocolo | O que observar no log |
|---|---|
| **TCP/IP** | IP/porta de origem e destino, direção, volume, conexões incomuns (portas altas, destinos externos inesperados) |
| **DNS** | Domínios consultados, consultas a domínios recém-criados/aleatórios, volume anormal de consultas |
| **HTTP** | URL, método, código de resposta, User-Agent, downloads de executáveis/scripts |
| **SMTP** | Remetente, destinatário, assunto, anexos, indícios de phishing/spoofing |

---

## 🗺️ 10. Roadmap

- [ ] Adicionar **Host 2 Linux** (agente via `curl`/pacote `.deb` ou `.rpm`) e monitorar `auth.log`/`secure`
- [ ] Instalar **Sysmon** no Windows e integrar ao Wazuh para telemetria de processos e rede
- [ ] Habilitar e documentar **FIM** em diretórios críticos
- [ ] Revisar relatórios (*Generate report*) e documentar um relatório diário de turno
- [ ] Preencher a regra (`rule.id`) e a classificação final do caso SOC-20261010-001
- [ ] Estudar para **Security+** e **ISO 27001 Foundation** (certificações desejáveis da vaga)

---

## 🧠 11. Competências Demonstradas

**Técnicas:** operação de SIEM/EDR, triagem por severidade, leitura de logs Windows, MITRE ATT&CK, FIM, instalação e validação de agentes, coleta de evidências, noções de TCP/IP, DNS, HTTP e SMTP.

**Comportamentais:** disciplina operacional (seguir runbooks), comunicação técnica escrita clara, organização e rastreabilidade, aprendizado contínuo (laboratório autônomo) e atuação sob supervisão com escalonamento adequado.

---

## ⚠️ Aviso

Laboratório com fins educacionais, executado em ambiente próprio e controlado. Credenciais, chaves de registro e hostnames reais **não** devem ser versionados neste repositório.
