# 🐢 SAP | Lentidão na Transação ZVDA

> **Categoria:** Processos e Procedimentos  
> **Ambiente:** SLATAM-BR · SAP S/4HANA  
> **Urgência típica:** Severe Disruption  
> **Última atualização:** Março/2026

---

## 📌 Identificação do Problema

| Campo | Detalhe |
|---|---|
| **Sintoma** | Lentidão ou travamento exclusivo na transação ZVDA |
| **Comportamento** | Sistema exibe popup "Interromper aplicação" durante carregamento |
| **Impacto** | Emissão de ordens ZVDA bloqueada; demais transações funcionam normalmente |
| **Escopo típico** | Um ou mais usuários da mesma unidade/filial |

---

## 🔍 Diagnóstico — Passo a Passo

### 1. Confirmar isolamento do problema

- [ ] Verificar se o problema ocorre com **outro usuário** da mesma unidade
- [ ] Verificar se o problema ocorre em **outra máquina**
- [ ] Confirmar que outras transações SAP funcionam normalmente

> ✅ Se o problema se repete com outros usuários → o problema está **no servidor/banco**, não na máquina do usuário.

---

### 2. SM50 / SM66 — Processos de trabalho

```
Transação: SM50 (local) ou SM66 (global)
```

- Verificar se há **work processes** com status `Running` por tempo elevado
- Identificar processos relacionados à ZVDA
- Checar se há processos em estado `Stopped` ou `Ended` inesperadamente

---

### 3. SM12 — Verificação de Locks

```
Transação: SM12
```

- Verificar entradas de lock em aberto relacionadas à ZVDA
- Identificar usuário/sessão que está segurando o lock
- Se necessário, **liberar o lock** com cautela após confirmar que a sessão está inativa

---

### 4. ST05 / SE30 — Performance Trace

```
Transação: ST05 (SQL Trace) ou SE30 (Runtime Analysis)
```

- Ativar trace para o usuário afetado
- Executar a ZVDA e reproduzir o problema
- Analisar:
  - Queries SQL com tempo elevado
  - Funções ABAP com alto consumo
  - Leituras de tabela sem índice (Full Table Scan)

---

### 5. ST22 — Dumps ABAP

```
Transação: ST22
```

- Verificar se há **dumps ABAP** recentes relacionados à ZVDA
- Analisar a causa raiz do dump (timeout, memória, loop infinito)

---

### 6. SM21 — Log do Sistema

```
Transação: SM21
```

- Verificar erros no log do sistema no período em que a lentidão ocorreu
- Filtrar por mensagens de erro críticas

---

### 7. DB02 / ST04 — Saúde do Banco de Dados

```
Transação: DB02 ou ST04
```

- Verificar se há tabelas utilizadas pela ZVDA com **estatísticas desatualizadas**
- Checar espaço em disco e performance geral do banco

---

## 💬 Resposta Padrão ao Usuário

```
Olá, [NOME DO USUÁRIO]! Tudo bem?

Recebemos seu chamado referente à lentidão na transação ZVDA.
Já iniciamos a análise no ambiente SLATAM-BR SAP S4-HANA.

Identificamos que o problema está ocorrendo de forma consistente
na transação. Nossa equipe está verificando os processos ativos
no servidor e possíveis locks que possam estar impactando o desempenho.

Em breve retornaremos com uma atualização.
Caso a situação se agrave, por favor entre em contato pelo suporte.

Atenciosamente,
Equipe de Suporte SAP
```

---

## ✅ Critérios de Resolução

- [ ] Causa raiz identificada e documentada
- [ ] Lentidão resolvida e ZVDA funcionando normalmente
- [ ] Usuário confirmou normalização
- [ ] Solução registrada no chamado antes de fechar

---

## 📎 Referências

- Chamado de origem: **SAP-INFRA-LENTIDAO/ALTO PROCESSAMENTO NO SERVIDOR**
- Usuário que originou o procedimento: Marilandes Rozendo — Filial Republica-SP (2500)
- Ambiente: `SLATAM-BR_SAP-S4-HANA`

---

> 💡 **Dica:** Caso o problema persista após todas as verificações, escalar para o time de Basis com os outputs das transações SM50, ST05 e ST22 em mãos.
