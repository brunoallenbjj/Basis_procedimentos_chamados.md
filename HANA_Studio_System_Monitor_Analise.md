# Procedimento de Análise — SAP HANA Studio: System Monitor

> **Categoria:** SAP HANA Administration  
> **Ferramenta:** SAP HANA Studio  
> **Versão do documento:** 1.0  
> **Última atualização:** 14/03/2026  

---

## Sumário

1. [Visão Geral](#1-visão-geral)
2. [Acesso ao System Monitor](#2-acesso-ao-system-monitor)
3. [Entendendo as Colunas do System Monitor](#3-entendendo-as-colunas-do-system-monitor)
4. [Interpretando o Operational State](#4-interpretando-o-operational-state)
5. [Interpretando os Alertas](#5-interpretando-os-alertas)
6. [Análise de Disco](#6-análise-de-disco)
7. [Análise de Memória](#7-análise-de-memória)
8. [Análise de CPU](#8-análise-de-cpu)
9. [O que mais explorar no HANA Studio](#9-o-que-mais-explorar-no-hana-studio)
10. [Roteiro de Análise Diária](#10-roteiro-de-análise-diária)
11. [Tabela de Referência Rápida — Sinais de Alerta](#11-tabela-de-referência-rápida--sinais-de-alerta)
12. [Queries SQL Úteis para Diagnóstico](#12-queries-sql-úteis-para-diagnóstico)

---

## 1. Visão Geral

O **System Monitor** do SAP HANA Studio é o painel central de observabilidade de todos os sistemas HANA registrados no landscape. Ele permite ao administrador visualizar, em uma única tela, o estado operacional, alertas ativos e métricas de recursos (disco, memória e CPU) de múltiplos sistemas simultaneamente.

**Quando usar:**
- Análise de saúde do ambiente (health check)
- Triagem inicial de incidentes
- Monitoramento preventivo de capacidade
- Verificação rápida após janelas de manutenção

---

## 2. Acesso ao System Monitor

1. Abra o **SAP HANA Studio**
2. Na perspectiva **SAP HANA Administration Console**, clique em:
   ```
   Window → Open Perspective → SAP HANA Administration Console
   ```
3. No menu superior, acesse:
   ```
   SAP HANA → System Monitor
   ```
   ou pressione `Ctrl + Shift + S` (atalho padrão em alguns ambientes)

4. A tela exibirá todos os sistemas cadastrados no **Systems Panel** à esquerda

> **Dica:** É possível configurar o intervalo de atualização automática clicando no ícone de relógio (⏱️) no canto superior direito do System Monitor. Recomenda-se **30 a 60 segundos** para ambientes produtivos.

---

## 3. Entendendo as Colunas do System Monitor

### 3.1 System ID

Identificador único do sistema HANA no formato:

```
SID@Hostname
```

Exemplos:
- `XS1@S01` → Sistema com SID "XS1" no host "S01"
- `SYSTEMDB@S01` → Banco de dados de sistema (System DB) no host S01
- `SH2@S02 (SAPHANADB)` → Tenant com nome descritivo entre parênteses

> **Atenção:** Sistemas com o prefixo `SYSTEMDB` são os bancos administrativos do MDC (Multi-Database Container). Sempre verifique o SYSTEMDB antes dos tenants.

---

### 3.2 Operational State

Estado operacional atual do sistema. Consulte a [Seção 4](#4-interpretando-o-operational-state) para detalhes.

---

### 3.3 Alerts

Número e severidade dos alertas ativos. Consulte a [Seção 5](#5-interpretando-os-alertas) para detalhes.

---

### 3.4 Data Disk (GB)

**Formato:** `Utilizado / Total`

Representa o espaço consumido e total do **volume de dados** do HANA (arquivos `.dat` / data volumes). Este volume armazena os dados persistidos em disco (savepoints e snapshots).

| Faixa de Uso | Status | Ação |
|---|---|---|
| < 60% | ✅ Normal | Monitoramento de rotina |
| 60% – 75% | 🟡 Atenção | Planejar expansão ou limpeza |
| 75% – 85% | 🟠 Alerta | Acionar time de infraestrutura |
| > 85% | 🔴 Crítico | Ação imediata necessária |

---

### 3.5 Log Disk (GB)

**Formato:** `Utilizado / Total`

Espaço consumido pelo **volume de redo log** (transações). O HANA grava todos os commits aqui antes de persistir nos dados. Se este volume encher, o banco para de aceitar escritas.

> ⚠️ **Criticidade elevada:** Um Log Disk cheio resulta em **parada total de operações de escrita** no banco. Monitorar com prioridade máxima.

---

### 3.6 Trace Disk (GB)

**Formato:** `Utilizado / Total`

Espaço ocupado pelos **arquivos de trace e diagnóstico** (logs internos do HANA, core dumps, arquivos de suporte). Normalmente cresce lentamente, mas pode crescer rapidamente em situações de falhas repetidas.

**Limpeza de trace:**
```
SAP HANA Studio → Administration → Diagnosis Files → Delete old files
```

---

### 3.7 Database Resident Memory

Memória RAM efetivamente **alocada e residente** para os dados do banco (tabelas em memória, índices, caches). É o principal indicador do consumo de memória do HANA.

---

### 3.8 System Resident Memory

Memória total **residente no sistema operacional** usada pelo processo HANA (inclui memória do banco + buffers + overhead do processo).

---

### 3.9 Used Memory (GB)

Memória total **em uso** pelo HANA, incluindo alocações que ainda não foram liberadas pelo SO. Útil para análise de tendência de crescimento.

---

### 3.10 CPU (%)

Percentual de CPU consumido pelo sistema HANA no momento da leitura. Valores altos por períodos prolongados indicam carga elevada.

| CPU % | Interpretação |
|---|---|
| 0% – 30% | Normal |
| 30% – 60% | Carga moderada (verificar jobs/queries em execução) |
| 60% – 80% | Carga alta (investigar queries de longa duração) |
| > 80% | Crítico — investigar imediatamente |

---

## 4. Interpretando o Operational State

### 🟢 All services started
Todos os serviços HANA estão operacionais. Estado desejado para sistemas produtivos.

### 🟡 'sapstartsrv' service not started
O serviço **SAP Host Agent** (`sapstartsrv`) não está em execução neste host. Este serviço é responsável pelo controle remoto dos processos SAP.

**Impacto:**
- Impossibilidade de iniciar/parar serviços remotamente via Studio ou SAPControl
- Monitoramento limitado pelo HANA Studio
- O banco de dados pode estar funcionando normalmente, mas sem controle administrativo remoto

**Ação:**
```bash
# No servidor afetado (usuário root ou <sid>adm):
/usr/sap/hostctrl/exe/saphostexec -restart
# ou
service sapinit restart
```

### 🔷 System status cannot be determined
O HANA Studio **não consegue se comunicar** com o sistema. O ícone losango cinza/azul indica falha de conectividade.

**Possíveis causas:**
1. Host desligado ou indisponível
2. Serviço HANA parado (`indexserver`, `nameserver`)
3. Firewall bloqueando a porta `3<instance>15` (ex: 30015 para instância 00)
4. SAP Host Agent parado no host remoto
5. Credenciais de acesso inválidas no Studio

**Diagnóstico:**
```bash
# Testar conectividade de rede:
ping <hostname>
telnet <hostname> 30015

# Verificar status do HANA no servidor:
su - <sid>adm
HDB info
HDB version
```

---

## 5. Interpretando os Alertas

Os alertas são gerados pelo **HANA Statistics Server** com base em thresholds configurados. São classificados em quatro níveis:

### 🔴 HIGH Priority (Vermelho)
Situação crítica. Pode impactar disponibilidade ou performance significativamente.

**Exemplos comuns:**
- Falta de espaço em disco (Data, Log ou Trace)
- Memória acima do limite configurado
- Serviço com falha ou reinicializações frequentes
- Backup não executado dentro do SLA

### 🟡 MEDIUM Priority (Amarelo/Laranja)
Situação que requer atenção, mas não é emergência imediata.

**Exemplos comuns:**
- Estatísticas de tabelas desatualizadas
- Conexões próximas ao limite
- Certificados SSL próximos ao vencimento

### 🔵 LOW Priority (Azul/Verde)
Alertas informativos ou de baixo impacto.

**Exemplos comuns:**
- Parâmetros com valores fora do recomendado
- Configurações não otimizadas

### ⚪ The system is running without alerts
Nenhum alerta ativo. Estado ideal.

---

**Como ver o detalhe dos alertas:**

1. Clique duplo no sistema desejado no System Monitor
2. Navegue até a aba **Alerts**
3. Clique em cada alerta para ver descrição, valor atual, threshold e recomendação

---

## 6. Análise de Disco

### Checklist de Disco

- [ ] Data Disk < 75% em todos os sistemas
- [ ] Log Disk < 60% em todos os sistemas
- [ ] Trace Disk < 50% em todos os sistemas
- [ ] Nenhum sistema com alerta de disco ativo

### Como verificar detalhes de disco pelo Studio

1. Clique duplo no sistema → aba **Configuration**
2. Em **Disk Usage**, visualize os volumes individualmente
3. Para ver histórico, acesse: `Performance → Disk` (se disponível na versão)

### Query SQL para verificar uso de disco

```sql
-- Verificar uso dos volumes de dados
SELECT
    HOST,
    VOLUME_ID,
    USED_SIZE / 1024 / 1024 / 1024 AS USED_GB,
    TOTAL_SIZE / 1024 / 1024 / 1024 AS TOTAL_GB,
    ROUND(USED_SIZE * 100.0 / TOTAL_SIZE, 2) AS PCT_USED
FROM M_VOLUME_FILES
ORDER BY PCT_USED DESC;
```

---

## 7. Análise de Memória

O HANA é um banco **in-memory** — a memória é seu recurso mais crítico.

### Conceitos importantes

| Conceito | Descrição |
|---|---|
| **Global Allocation Limit** | Limite máximo de memória que o HANA pode alocar |
| **Resident Memory** | Memória efetivamente em uso nas páginas de RAM |
| **Heap Memory** | Memória alocada dinamicamente pelos serviços |
| **Column Store** | Parte da memória usada pelas tabelas colunares |

### Como verificar memória pelo Studio

1. Clique duplo no sistema → aba **Memory**
2. Visualize o gráfico de uso ao longo do tempo
3. Verifique serviço por serviço (indexserver, xsengine, etc.)

### Query SQL para análise de memória

```sql
-- Memória por serviço
SELECT
    HOST,
    SERVICE_NAME,
    ROUND(PHYSICAL_MEMORY_SIZE / 1024 / 1024 / 1024, 2) AS PHYSICAL_MEM_GB,
    ROUND(VIRTUAL_MEMORY_SIZE / 1024 / 1024 / 1024, 2) AS VIRTUAL_MEM_GB
FROM M_SERVICE_MEMORY
ORDER BY PHYSICAL_MEMORY_SIZE DESC;

-- Maior consumo de memória por tabela
SELECT TOP 20
    SCHEMA_NAME,
    TABLE_NAME,
    ROUND(MEMORY_SIZE_IN_TOTAL / 1024 / 1024, 2) AS SIZE_MB
FROM M_CS_TABLES
ORDER BY MEMORY_SIZE_IN_TOTAL DESC;
```

---

## 8. Análise de CPU

### Como verificar CPU pelo Studio

1. Clique duplo no sistema → aba **Performance**
2. Acesse **Top Queries** para identificar consultas pesadas
3. Verifique **Active Threads** para ver o que está em execução no momento

### Query SQL para análise de CPU

```sql
-- Queries ativas com alto consumo de CPU
SELECT
    CONNECTION_ID,
    USER_NAME,
    APPLICATION_NAME,
    STATEMENT_STATUS,
    ROUND(CPU_TIME / 1000000, 2) AS CPU_SECONDS,
    STATEMENT_STRING
FROM M_ACTIVE_STATEMENTS
ORDER BY CPU_TIME DESC;

-- Top statements por CPU (histórico)
SELECT TOP 20
    USER_NAME,
    ROUND(SUM(CPU_TIME) / 1000000, 2) AS TOTAL_CPU_SEC,
    COUNT(*) AS EXECUTIONS,
    STATEMENT_HASH
FROM M_SQL_PLAN_CACHE
GROUP BY USER_NAME, STATEMENT_HASH
ORDER BY TOTAL_CPU_SEC DESC;
```

---

## 9. O que mais explorar no HANA Studio

O HANA Studio oferece muito além do System Monitor. Veja as principais áreas de análise disponíveis:

---

### 9.1 Administration Console (por sistema)

Ao clicar duplo em qualquer sistema, abre-se o **Administration Console** com as seguintes abas:

#### 📋 Overview
- Visão consolidada de saúde do sistema
- Links rápidos para as demais abas
- Resumo de alertas e uso de recursos

#### 🔔 Alerts
- Lista completa de alertas ativos e histórico
- Nível de severidade, descrição e recomendações
- Possibilidade de criar alertas customizados

#### 🔧 Configuration
- Parâmetros do sistema HANA (`.ini` files)
- Modificação de configurações com trilha de auditoria
- Comparação entre configurações ativas e recomendadas

#### 📊 Performance
- **Threads:** Threads ativas por serviço
- **Sessions:** Sessões abertas e duração
- **SQL Console:** Editor SQL integrado
- **Job Progress:** Andamento de operações longas (re-org, backup, etc.)
- **Blocked Transactions:** Deadlocks e bloqueios

#### 💾 Volumes
- Estado dos volumes de dados e log
- Histórico de crescimento
- Operações de data area management

#### 🗄️ Backup
- Status do último backup (Full, Differential, Log)
- Catálogo de backups disponíveis
- Configuração da política de backup
- Iniciar backup manual

#### 📁 Diagnosis Files
- Arquivos de trace e log gerados pelo HANA
- Possibilidade de download e análise
- Limpeza de arquivos antigos
- Criação de pacotes de suporte (support packages) para SAP

#### 👥 Security
- Gerenciamento de usuários e roles
- Políticas de senha
- Auditoria de acessos

---

### 9.2 SQL Console

Acessível pelo menu de contexto de qualquer sistema (botão direito → **Open SQL Console**) ou pelo atalho `Ctrl + Alt + S`.

**Recursos disponíveis:**
- Execução de queries ad-hoc
- Visualização de planos de execução (`Explain Plan`)
- Histórico de comandos executados
- Exportação de resultados para CSV/Excel

**Views de sistema mais utilizadas:**

| View | Descrição |
|---|---|
| `M_ACTIVE_STATEMENTS` | Queries em execução no momento |
| `M_SERVICE_MEMORY` | Memória por serviço |
| `M_VOLUME_FILES` | Arquivos de volume de dados |
| `M_BACKUP_CATALOG` | Catálogo de backups |
| `M_LANDSCAPE_HOST_CONFIGURATION` | Configuração dos hosts |
| `M_SYSTEM_OVERVIEW` | Visão geral do sistema |
| `M_CS_TABLES` | Tabelas column store e uso de memória |
| `M_EXPENSIVE_STATEMENTS` | Histórico de queries caras |
| `M_BLOCKED_TRANSACTIONS` | Transações bloqueadas |

---

### 9.3 Navigator (Catálogo)

Permite explorar a estrutura do banco:

```
Systems → <Sistema> → Catalog → <Schema> → Tables / Views / Procedures
```

- Visualizar definição de tabelas e índices
- Executar `SELECT TOP 1000` diretamente
- Ver estatísticas de tabelas (número de linhas, tamanho)
- Acesso a stored procedures e funções

---

### 9.4 Modeler (apenas versões com XS Classic)

Para ambientes com SAP HANA XS Classic:
- Criação e edição de **Calculation Views**
- **Analytic Views** e **Attribute Views**
- Debugging de modelos de dados

---

### 9.5 Quick Launch (Atalhos Úteis)

| Ação | Caminho no Studio |
|---|---|
| Verificar status de backup | Administration → Backup |
| Analisar queries lentas | Performance → Expensive Statements |
| Ver sessões ativas | Performance → Sessions |
| Verificar bloqueios | Performance → Blocked Transactions |
| Ajustar parâmetros | Administration → Configuration |
| Baixar logs de trace | Administration → Diagnosis Files |
| Gerenciar usuários | Security → Users |

---

## 10. Roteiro de Análise Diária

Use este roteiro como checklist de análise preventiva diária (recomendado: executar 1x por dia, preferencialmente pela manhã).

### Passo 1 — Visão Geral no System Monitor

- [ ] Abrir o System Monitor
- [ ] Identificar sistemas com estado diferente de **"All services started"**
- [ ] Identificar sistemas com alertas **HIGH** ou **MEDIUM**
- [ ] Verificar sistemas **inacessíveis** (System status cannot be determined)

---

### Passo 2 — Análise de Disco

- [ ] Verificar coluna **Data Disk** — nenhum sistema acima de 75%
- [ ] Verificar coluna **Log Disk** — nenhum sistema acima de 60%
- [ ] Verificar coluna **Trace Disk** — nenhum sistema acima de 50%
- [ ] Em sistemas próximos dos limites, abrir o detalhe e planejar ação

---

### Passo 3 — Análise de Alertas

- [ ] Para cada sistema com alerta HIGH, abrir **Administration → Alerts**
- [ ] Documentar o alerta, causa raiz identificada e ação tomada
- [ ] Verificar se alertas MEDIUM anteriores foram resolvidos

---

### Passo 4 — Verificação de Backup

- [ ] Abrir **Administration → Backup** em cada sistema produtivo
- [ ] Confirmar que o backup mais recente foi concluído com sucesso
- [ ] Verificar data/hora do último Full Backup e Log Backup
- [ ] Alertar time responsável se algum backup estiver atrasado

---

### Passo 5 — Performance Spot Check

- [ ] Verificar CPU (%) — sistemas acima de 60% merecem investigação
- [ ] Acessar **Performance → Threads** em sistemas com CPU alta
- [ ] Executar query em `M_EXPENSIVE_STATEMENTS` para identificar top queries do dia

---

### Passo 6 — Documentação

- [ ] Registrar no sistema de chamados qualquer anomalia identificada
- [ ] Atualizar planilha/dashboard de capacidade com valores atuais de disco e memória
- [ ] Escalar para o time de infraestrutura situações que exijam expansão de recursos

---

## 11. Tabela de Referência Rápida — Sinais de Alerta

| Indicador | Limite de Atenção | Limite Crítico | Ação Recomendada |
|---|---|---|---|
| Data Disk | > 75% | > 85% | Limpeza de dados / expansão de volume |
| Log Disk | > 60% | > 75% | Verificar política de backup de log / expansão |
| Trace Disk | > 50% | > 70% | Limpar arquivos de trace antigos |
| CPU (%) | > 60% | > 80% | Investigar queries/jobs em execução |
| Operational State | sapstartsrv not started | System status cannot be determined | Reiniciar serviço / verificar host |
| Alertas | MEDIUM ativos | HIGH ativos | Análise imediata e abertura de chamado |

---

## 12. Queries SQL Úteis para Diagnóstico

### 12.1 Visão geral do sistema

```sql
SELECT * FROM M_SYSTEM_OVERVIEW;
```

### 12.2 Uso de memória consolidado

```sql
SELECT
    HOST,
    ROUND(FREE_PHYSICAL_MEMORY / 1024 / 1024 / 1024, 2) AS FREE_GB,
    ROUND(USED_PHYSICAL_MEMORY / 1024 / 1024 / 1024, 2) AS USED_GB,
    ROUND(TOTAL_CPU_USER_TIME, 2) AS CPU_USER_PCT
FROM M_HOST_RESOURCE_UTILIZATION;
```

### 12.3 Último backup por tipo

```sql
SELECT
    ENTRY_TYPE_NAME,
    MAX(UTC_END_TIME) AS LAST_BACKUP,
    STATE_NAME,
    BACKUP_SIZE / 1024 / 1024 / 1024 AS SIZE_GB
FROM M_BACKUP_CATALOG
WHERE STATE_NAME = 'successful'
GROUP BY ENTRY_TYPE_NAME, STATE_NAME, BACKUP_SIZE
ORDER BY LAST_BACKUP DESC;
```

### 12.4 Transações bloqueadas

```sql
SELECT
    BLOCKED_CONNECTION_ID,
    BLOCKING_CONNECTION_ID,
    WAIT_LOCK_NAME,
    WAIT_SECONDS
FROM M_BLOCKED_TRANSACTIONS
ORDER BY WAIT_SECONDS DESC;
```

### 12.5 Top 10 tabelas por tamanho em memória

```sql
SELECT TOP 10
    SCHEMA_NAME,
    TABLE_NAME,
    ROUND(MEMORY_SIZE_IN_TOTAL / 1024 / 1024, 2) AS SIZE_MB,
    RECORD_COUNT
FROM M_CS_TABLES
ORDER BY MEMORY_SIZE_IN_TOTAL DESC;
```

### 12.6 Queries de longa duração (ativas agora)

```sql
SELECT
    START_TIME,
    SECONDS_BETWEEN(START_TIME, NOW()) AS DURATION_SEC,
    USER_NAME,
    APPLICATION_NAME,
    STATEMENT_STRING
FROM M_ACTIVE_STATEMENTS
WHERE SECONDS_BETWEEN(START_TIME, NOW()) > 30
ORDER BY DURATION_SEC DESC;
```

---

*Documento mantido pela equipe de Basis / SAP Administration.*  
*Para dúvidas ou contribuições, contate o responsável pela base de conhecimento.*
