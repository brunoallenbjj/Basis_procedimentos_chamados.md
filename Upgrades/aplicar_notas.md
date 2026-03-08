# 📘 SAP Basis Runbook

## Aplicação de Nova SAP Note

---

## 🎯 Objetivo

Este procedimento descreve o passo a passo para **download, verificação e implementação de uma SAP Note** no sistema utilizando o SAP Note Assistant.

As SAP Notes são utilizadas para:

* Correção de **bugs**
* Ajustes de **segurança**
* Melhorias de **performance**
* Correções recomendadas pela SAP

---

## ⚙️ Pré-requisitos

Antes de iniciar a aplicação da nota, valide:

* Acesso autorizado de **SAP Basis**
* Sistema conectado ao **SAP Support Portal**
* Nota válida para a **versão do sistema**
* Ordem de transporte disponível
* Aplicação preferencialmente em **ambiente DEV ou QAS**
* Janela de manutenção aprovada (se necessário)

---

## 🧭 Passo a Passo

### 1️⃣ Acessar o SAP Note Assistant

Acesse a transação:

```
SNOTE
```

Caminho:

```
SAP Easy Access → Tools → ABAP Workbench → Utilities → SAP Note Assistant
```

---

### 2️⃣ Baixar a SAP Note

Dentro da transação **SNOTE**:

```
Goto → Download SAP Note
```

1. Inserir o **número da SAP Note**
2. Executar (**F8**)

O sistema irá realizar o download da nota diretamente do repositório SAP.

---

### 3️⃣ Verificar consistência da Nota

Após o download:

1. Localize a nota na lista
2. Abra a nota
3. Clique em:

```
Check SAP Note
```

Essa etapa verifica:

* Dependências
* Conflitos com outras notas
* Objetos afetados

Caso existam **notas pré-requisito**, elas devem ser implementadas primeiro.

---

### 4️⃣ Implementar a Nota

Se a verificação estiver correta:

1. Selecione a nota
2. Clique em:

```
Implement SAP Note
```

Durante o processo o sistema pode solicitar:

* criação de **ordem de transporte**
* confirmação de objetos modificados

Selecione ou crie uma **ordem de transporte apropriada**.

---

### 5️⃣ Ativar Objetos

Caso existam objetos pendentes:

```
Activate Objects
```

Certifique-se que **todos os objetos estejam ativos** após a implementação.

---

### 6️⃣ Verificar Logs

Após a implementação:

1. Acesse o **log da nota**
2. Verifique se ocorreram erros ou warnings

---

### 7️⃣ Validar a Correção

Realize:

* testes funcionais
* validação do cenário corrigido
* confirmação com o usuário solicitante

---

## 🔎 Troubleshooting

### Nota não pode ser implementada

Possíveis causas:

* Dependência de outras notas
* Conflito com modificações Z
* Objetos já modificados manualmente

Solução:

```
Check SAP Note → Verificar prerequisitos
```

---

### Objeto não ativa

Verificar na transação:

```
SE80
```

ou

```
SE38
```

Ativar manualmente o objeto.

---

## 📋 Boas Práticas SAP Basis

* Aplicar sempre primeiro em **DEV**
* Transportar para **QAS → PRD**
* Registrar implementação no **chamado**
* Documentar número da **ordem de transporte**
* Validar dependências antes da implementação

---

## 📚 Referências

* SAP Note Assistant Documentation
* SAP Support Portal

---

## 🤝 Contribuições

Contribuições são bem-vindas!

Caso queira melhorar este procedimento:

1. Faça um **fork do repositório**
2. Crie uma **branch**
3. Submeta um **pull request**

---

**Autor:** Equipe SAP Basis
**Categoria:** Runbook / Troubleshooting
**Última atualização:** 2026
