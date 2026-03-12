# 🔍 SAP Basis — Análise de Lentidão em Ambiente Fiori

> **Categoria:** Performance | **Plataforma:** SAP Fiori / S4HANA / ECC  
> **Autor:** Bruno Almeida | **Última atualização:** 2026-03-10

---

## 📋 Índice

1. [Contexto do Incidente](#1-contexto-do-incidente)
2. [Coleta de Informações com o Usuário](#2-coleta-de-informações-com-o-usuário)
3. [Etapa 1 — Work Processes (SM50 / SM66)](#3-etapa-1--work-processes-sm50--sm66)
4. [Etapa 2 — Logs e Traces (SM21 / ST05 / IWFND)](#4-etapa-2--logs-e-traces-sm21--st05--iwfnd)
5. [Etapa 3 — Gateway e OData (Fiori)](#5-etapa-3--gateway-e-odata-fiori)
6. [Etapa 4 — Banco de Dados (ST04 / DB02)](#6-etapa-4--banco-de-dados-st04--db02)
7. [Etapa 5 — Recursos do Servidor (OS07 / AL08)](#7-etapa-5--recursos-do-servidor-os07--al08)
8. [Etapa 6 — Workload Monitor (ST03N)](#8-etapa-6--workload-monitor-st03n)
9. [Etapa 7 — ICM e Fiori Launchpad (SMICM / SICF)](#9-etapa-7--icm-e-fiori-launchpad-smicm--sicf)
10. [Fluxo Resumido de Diagnóstico](#10-fluxo-resumido-de-diagnóstico)

---

## 1. Contexto do Incidente

| Campo | Detalhe |
|---|---|
| **Sintoma relatado** | Página sem resposta ao criar NFSE (Nota Fiscal de Serviço Eletrônica) |
| **Aplicação** | SAP Fiori — transação `NotaFiscal-create` |
| **Ambiente** | Produtivo / Qas / Dev |
| **Evidência** | Browser exibindo popup *"Página sem resposta"* — aba travada aguardando resposta do servidor |
| **Possíveis causas** | Lentidão no backend SAP, no Gateway/OData, na rede ou recursos do servidor esgotados |

---

## 2. Coleta de Informações com o Usuário

Antes de acessar qualquer transação, colete o contexto com quem abriu o chamado:

- [ ] O problema ocorre **somente nessa transação** ou em todas as telas Fiori?
- [ ] **Quando começou?** Foi após alguma atualização, transporte ou mudança no ambiente?
- [ ] O problema é **recorrente ou esporádico?**
- [ ] **Outros usuários** estão com o mesmo problema?
- [ ] A lentidão ocorre na **abertura da tela**, no **salvamento** ou em alguma ação específica?
- [ ] O usuário está em **home office (VPN)** ou rede corporativa?

> 💡 **Dica:** Se apenas um usuário está com o problema e os demais não, o foco inicial deve ser na estação/rede do usuário e em seus locks de sessão no SAP.

---

## 3. Etapa 1 — Work Processes (SM50 / SM66)

Verifique a saúde dos processos de trabalho do sistema.

| Transação | Escopo |
|---|---|
| `SM50` | Work Processes do servidor de aplicação **local** |
| `SM66` | Work Processes de **todos os servidores** (visão global) |

### O que observar

| Status | Significado | Ação |
|---|---|---|
| `Running` por > 30s | Processo travado / longa execução | Identificar usuário e programa; avaliar cancelamento |
| `PRIV` | Processo em memory swap | Crítico — verificar memória RAM do servidor |
| `Waiting` em grande quantidade | Fila de processos acumulada | Verificar carga total do sistema |
| Todos WPs `DIA` ocupados | Usuários sem atendimento | Aumentar WPs ou eliminar processos travados |

---

## 4. Etapa 2 — Logs e Traces (SM21 / ST05 / IWFND)

### SM21 — System Log

```
Menu: Tools > Administration > Monitor > System Log
Transação: SM21
```

- Filtrar pelo **horário exato** do incidente relatado
- Buscar erros críticos, dumps, avisos de memória ou timeout

### ST05 — SQL Trace

```
Transação: ST05
```

1. Selecione o usuário afetado
2. Ative o trace: **Activate Trace**
3. Peça para o usuário **reproduzir o problema**
4. Desative o trace: **Deactivate Trace**
5. Analise os statements SQL mais lentos (ordenar por `Exec. Time`)

> ⚠️ **Atenção:** Ative o trace somente no momento da reprodução para não gerar volume excessivo de dados.

### /IWFND/ERROR_LOG — Log de Erros do Gateway

```
Transação: /IWFND/ERROR_LOG
```

- Verificar erros recentes associados ao serviço `NOTAFISCAL` ou similar
- Observar códigos HTTP: `500`, `503`, `504` (timeout)

---

## 5. Etapa 3 — Gateway e OData (Fiori)

Específico para ambientes Fiori — o Gateway é o intermediário entre o browser e o backend SAP.

| Transação | Finalidade |
|---|---|
| `/IWFND/MAINT_SERVICE` | Verificar se o serviço OData está ativo e publicado |
| `/IWFND/ERROR_LOG` | Erros recentes do Gateway |
| `/IWFND/TRACES` | Ativar trace detalhado para o usuário/serviço com problema |

### O que buscar

- Timeout nas chamadas OData
- Serviços com erro HTTP `500`, `503` ou timeout
- Latência elevada nos requests (campo `Response Time`)
- Serviço desativado ou não publicado no sistema backend

---

## 6. Etapa 4 — Banco de Dados (ST04 / DB02)

### ST04 — Database Performance Monitor

```
Transação: ST04
```

| Indicador | Valor Normal | Alerta |
|---|---|---|
| Buffer Hit Ratio | > 95% | < 95% indica problema de memória/buffer |
| Logical Reads | Referência do ambiente | Pico anormal = queries sem índice |
| Physical Reads | Baixo | Alto = I/O de disco elevado |

### DB02 — Espaço e Estatísticas

```
Transação: DB02
```

- Verificar **tablespace** sem espaço livre (pode causar lentidão e erros)
- Verificar **estatísticas desatualizadas** nas tabelas críticas de NF

> 💡 **Dica HANA:** Se o banco for SAP HANA, utilizar a transação `DBACOCKPIT` para análise mais completa.

---

## 7. Etapa 5 — Recursos do Servidor (OS07 / AL08)

| Transação | Finalidade |
|---|---|
| `AL08` | Usuários logados em todos os servidores de aplicação |
| `OS07` | Monitor de Sistema Operacional (CPU, memória, swap) |
| `SM51` | Lista de servidores de aplicação ativos |

### Thresholds de atenção

| Recurso | Normal | Alerta | Crítico |
|---|---|---|---|
| CPU | < 60% | 60–80% | > 80% sustentado |
| Memória RAM | < 75% | 75–90% | > 90% |
| Swap | 0% | Qualquer uso | > 10% |

---

## 8. Etapa 6 — Workload Monitor (ST03N)

```
Transação: ST03N
Caminho: Tools > Administration > Monitor > Performance > Workload > ST03N
```

### Como usar

1. Selecione o **servidor** e o **período** do incidente
2. Compare o **tempo de resposta médio** com horários normais
3. Filtre pela transação/programa relacionado à NF (`NOTAFISCAL`, `J_1B*`, etc.)
4. Verifique se houve pico de carga no horário relatado

### Métricas importantes

| Métrica | Descrição |
|---|---|
| `Response Time` | Tempo total de resposta ao usuário |
| `CPU Time` | Tempo de processamento puro |
| `Wait Time` | Tempo aguardando WP disponível |
| `DB Request Time` | Tempo gasto em chamadas ao banco |

---

## 9. Etapa 7 — ICM e Fiori Launchpad (SMICM / SICF)

### SMICM — Internet Communication Manager

```
Transação: SMICM
Menu: Goto > Threads
```

| O que observar | Problema |
|---|---|
| Threads HTTP com status `BUSY` por tempo prolongado | Gargalo no ICM |
| Fila de requests acumulada | Threads insuficientes para a demanda |
| Threads `KEEP_ALIVE` esgotando o pool | Configuração de keep-alive muito longa |

### SICF — Verificar serviços ativos

```
Transação: SICF
Caminho do serviço Fiori: /sap/bc/ui5_ui5
```

- Confirmar que o serviço Fiori está **ativo** (não desativado por acidente)
- Verificar serviços de NF específicos da empresa

---

## 10. Fluxo Resumido de Diagnóstico

```
Usuário reporta lentidão no Fiori
            │
            ▼
  Coletar contexto com o usuário
  (só ele? só essa tela? quando começou?)
            │
            ▼
  SM66 ──► WPs saturados ou em PRIV?
            │
            ▼
  /IWFND/ERROR_LOG ──► Erro no Gateway / OData?
            │
            ▼
  SMICM ──► Threads ICM esgotadas?
            │
            ▼
  ST04 / DBACOCKPIT ──► Banco sobrecarregado?
            │
            ▼
  OS07 ──► CPU / Memória no limite?
            │
            ▼
  ST03N ──► Workload anormal no horário?
            │
            ▼
  Identificar causa raiz → Aplicar correção → Documentar
```

---

## 📎 Referências e SAP Notes Relacionadas

| SAP Note | Descrição |
|---|---|
| `1797646` | Performance issues in Gateway / OData services |
| `2456119` | Fiori Launchpad — performance best practices |
| `2141696` | ICM thread configuration recommendations |
| `1931096` | SAP HANA — performance analysis with DBACOCKPIT |

---

> 📝 **Observação:** Sempre que possível, colete os dados das transações acima no **horário exato do incidente** ou logo após. Evidências coletadas com muita defasagem de tempo podem não refletir o estado do sistema durante o problema.
