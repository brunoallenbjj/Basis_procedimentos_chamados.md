# SAP Basis Runbook — Desbloqueio de Chave (Lock) no SAP

## Objetivo
Padronizar o procedimento de análise e remoção de **locks (chave presa)** no SAP quando um usuário não consegue prosseguir em uma transação devido a bloqueio de objeto/documento.

Exemplos de mensagens:


---

# Conceito Técnico

O SAP utiliza o **Enqueue Server** para controlar concorrência de acesso aos dados.

Quando um usuário acessa ou altera um documento, o sistema cria um **lock lógico** em memória para evitar alterações simultâneas.

Se a sessão for encerrada incorretamente (queda de energia, falha de rede, fechamento abrupto), o lock pode permanecer no sistema.

---

# Fluxo de Resolução


  
---

# 1. Identificação Inicial

Coletar no chamado:

| Informação | Exemplo |
|-------------|--------|
Usuário SAP | xxxxxx |
Documento | Ordem 4379423974234809238094 |
Transação | ZZ11 |
Ambiente | Produção |

---

# 2. Verificar Sessões Ativas

### Transação SM04


### Passos

1. Acessar **SM04**
2. Localizar o usuário informado
3. Verificar sessões abertas
4. Identificar a transação ativa

Exemplo:

| Usuário | Transação |
|--------|-----------|
XXXXX | zz11 |

### Possíveis cenários

| Situação | Ação |
|--------|------|
Usuário logado | Solicitar encerramento do SAP |
Sessão travada | Eliminar sessão |
Usuário não logado | Prosseguir para SM12 |

### Encerrar Sessão

Selecionar a sessão e clicar em: Eliminar sessão.




Aguardar aproximadamente **30 a 60 segundos**.

O lock normalmente é liberado automaticamente.

---

# 3. Verificar Locks do Sistema

### Transação SM12


  
### Passos

1. Acessar **SM12**
2. Filtrar pelo usuário
3. Executar busca

Campos importantes:

| Campo | Descrição |
|------|-----------|
User | Usuário responsável pelo lock |
Table | Tabela bloqueada |
Argument | Documento associado |
Lock Object | Objeto de enqueue |

---

# 4. Locks Mais Comuns por Módulo

| Módulo | Tabela | Documento |
|------|------|------|
SD | VBAK | Ordem de venda |
MM | EKKO | Pedido de compra |
MM | RBKP | Documento MIRO |
FI | BKPF | Documento contábil |

---

# 5. Verificar Atualizações Pendentes

Antes de remover qualquer lock manualmente.

### Transação SM13

  
### Passos

1. Filtrar pelo usuário
2. Verificar status

| Status | Significado |
|------|------|
Init | Update iniciado |
Auto | Update automático |
Error | Erro no update |

Se existir **update em erro**, analisar log antes de remover o lock.

---

# 6. Remoção Manual de Lock

Executar apenas se:

- Usuário não estiver logado
- Não existir update pendente
- Lock persistir após eliminação de sessão

### Passos

1. Acessar **SM12**
2. Selecionar lock
3. Clicar em: Delete


  
---

# 7. Validação Final

Após remoção do lock:

1. Solicitar que o usuário faça login novamente
2. Executar a transação novamente
3. Validar acesso ao documento

---

# 8. Causas Comuns de Locks

| Causa | Descrição |
|------|-----------|
Sessão paralela | Documento aberto em duas janelas |
Queda de energia | Sessão interrompida abruptamente |
Falha de rede | Conexão perdida |
Fechamento abrupto do SAP | Frontend encerrado sem commit |

---

# 9. Boas Práticas SAP Basis

- Sempre verificar **SM04 antes de SM12**
- Validar **SM13 antes de apagar locks**
- Evitar remoção de locks em processos críticos
- Confirmar se o usuário realmente saiu do sistema

---

# 10. Transações Utilizadas

| Transação | Função |
|------|------|
SM04 | Sessões de usuários |
SM12 | Locks do sistema |
SM13 | Atualizações pendentes |
SM66 | Processos ativos |
SM37 | Jobs batch |

---

# Responsável

Time SAP Basis


  
