# DER — Diagrama Entidade-Relacionamento
## Cereja — Sprint 2: Autenticação e Usuários

**Data:** 14/07/2026  
**Versão:** 1.0  
**Status:** ✅ Validado

---

## 📊 Visualização do Diagrama

```
┌─────────────────────────────┐
│         usuario             │
├─────────────────────────────┤
│ id (PK)                     │
│ email (UNIQUE)              │
│ senha                       │
│ nome                        │
│ telefone                    │
│ documento (UNIQUE)          │
│ data_criacao                │
│ ativo                       │
└──────────────┬──────────────┘
               │ (1,n)
               │
        ┌──────────────┐
        │usuario_role  │
        ├──────────────┤
        │usuario_id(FK)│
        │role_id (FK)  │
        └──────────────┘
               │
               │ (0,n)
               │
        ┌──────────────┐
        │    role      │
        ├──────────────┤
        │ id (PK)      │
        │ nome (UNIQUE)│
        │              │
        │ • CLIENTE    │
        │ • CONFEITEIRA│
        └──────────────┘
```

---

## 📋 Descrição das Entidades

### Entidade: Usuario

**Descrição:** Representa cada usuário do sistema (cliente ou confeiteira)

| Atributo     | Tipo         | Restrição             | Descrição              |
|--------------|--------------|-----------------------|------------------------|
| id           | BIGINT       | PK, AUTO_INCREMENT    | Identificador único    |
| email        | VARCHAR(255) | UNIQUE, NOT NULL      | Email para login       |
| senha        | VARCHAR(255) | NOT NULL              | Senha hasheada         |
| nome         | VARCHAR(255) | NOT NULL              | Nome completo          |
| telefone     | VARCHAR(20)  | NOT NULL              | Telefone               |
| documento    | VARCHAR(20)  | UNIQUE, NOT NULL      | CPF ou CNPJ            |
| data_criacao | TIMESTAMP    | DEFAULT NOW           | Data de criação        |
| ativo        | BOOLEAN      | DEFAULT TRUE          | Status (ativo/inativo) |

**Exemplo:**
```
ID: 1
Email: joao@email.com
Senha: $2a$10$...
Nome: João Silva
Telefone: 11987654321
Documento: 12345678901
Data Criação: 2026-07-14 10:30:00
Ativo: true
```

---

### Entidade: Role

**Descrição:** Tipos de acesso/perfil no sistema

| Atributo | Tipo        | Restrição             | Descrição           |
|----------|-------------|-----------------------|---------------------|
| id       | BIGINT      | PK, AUTO_INCREMENT    | Identificador único |
| nome     | VARCHAR(50) | UNIQUE, NOT NULL      | Nome da role        |

**Dados Padrão:**
```
ID: 1, Nome: CLIENTE
ID: 2, Nome: CONFEITEIRA
```

---

### Entidade: UsuarioRole (Associativa/Junção)

**Descrição:** Relaciona usuários com suas roles (um usuário pode ter múltiplas roles)

| Atributo   | Tipo   | Restrição        | Descrição               |
|------------|--------|------------------|-------------------------|
| usuario_id | BIGINT | PK (parte 1), FK | Referência para usuario |
| role_id    | BIGINT | PK (parte 2), FK | Referência para role    |

**Relacionamentos:**
- FK usuario_id → usuario.id (ON DELETE CASCADE)
- FK role_id → role.id (ON DELETE RESTRICT)

---

## 🔗 Relacionamentos

### Relacionamento 1: Usuario ↔ UsuarioRole

**Tipo:** Um para Muitos (1:N)  
**Cardinalidade:** (1, n)  
**Descrição:** Um usuário pode ter 0 ou mais roles

```
1 João (usuario_id=1)
├── CLIENTE (role_id=1)
└── CONFEITEIRA (role_id=2)

1 Maria (usuario_id=2)
└── CLIENTE (role_id=1)
```

**Regra de Integridade:**
- ON DELETE CASCADE: Se deletar usuário, suas roles são deletadas também

---

### Relacionamento 2: Role ↔ UsuarioRole

**Tipo:** Um para Muitos (1:N)  
**Cardinalidade:** (1, n)  
**Descrição:** Uma role pode ser atribuída a 1 ou mais usuários

```
1 CLIENTE (role_id=1)
├── João (usuario_id=1)
└── Maria (usuario_id=2)

1 CONFEITEIRA (role_id=2)
└── João (usuario_id=1)
```

**Regra de Integridade:**
- ON DELETE RESTRICT: Não permite deletar uma role se houver usuários com ela

---

## 📐 Propriedades do Relacionamento

| Propriedade      | Valor                    | Motivo                              |
|------------------|--------------------------|-------------------------------------|
| Tipo             | Muitos-para-Muitos (N:N) | Um usuário pode ter múltiplas roles |
| Tabela de Junção | usuario_role             | Implementa o relacionamento N:N     |
| Chave Primária   | (usuario_id, role_id)    | Garante unicidade                   |
| ON DELETE        | CASCADE                  | Limpa roles ao deletar usuário      |
| ON UPDATE        | CASCADE                  | Atualiza automaticamente            |

---

## ✅ Normalização

**Nível: 3ª Forma Normal (3NF)**

✅ **1FN** — Todos os valores são atômicos
- Sem listas ou valores compostos
- Cada campo tem um valor único

✅ **2FN** — Sem dependências parciais
- usuario_role tem chave composta (usuario_id, role_id)
- Ambos necessários para identificar o registro

✅ **3FN** — Sem dependências transitivas
- Nenhum campo depende indiretamente de outro
- Exemplo: nome não depende de email

---

## 🔄 Exemplos de Dados

### Cenário: João é cliente E confeiteira

**Tabela usuario:**
```
id | email                | nome       | telefone   | documento
1  | joao@email.com       | João Silva | 11987654321| 12345678901
```

**Tabela role:**
```
id | nome
1  | CLIENTE
2  | CONFEITEIRA
```

**Tabela usuario_role:**
```
usuario_id | role_id
1          | 1        ← João é CLIENTE
1          | 2        ← João é CONFEITEIRA
```

**Interpretação:** João tem 2 linhas na tabela usuario_role, uma para cada role.

---

### Cenário: Maria é apenas cliente

**Tabela usuario_role:**
```
usuario_id | role_id
2          | 1        ← Maria é CLIENTE
```

**Interpretação:** Maria tem apenas 1 linha, com a role CLIENTE.

---

## 🛠️ Queries Comuns

### Buscar todas as roles de um usuário

```sql
SELECT r.nome 
FROM role r
INNER JOIN usuario_role ur ON r.id = ur.role_id
WHERE ur.usuario_id = 1;
-- Resultado: CLIENTE, CONFEITEIRA (para João)
```

### Verificar se usuário é CLIENTE

```sql
SELECT COUNT(*) > 0 as isCliente
FROM usuario_role ur
INNER JOIN role r ON r.id = ur.role_id
WHERE ur.usuario_id = 1 AND r.nome = 'CLIENTE';
-- Resultado: true
```

### Listar todos os CLIENTES

```sql
SELECT DISTINCT u.id, u.nome, u.email
FROM usuario u
INNER JOIN usuario_role ur ON u.id = ur.usuario_id
INNER JOIN role r ON r.id = ur.role_id
WHERE r.nome = 'CLIENTE';
```

---

## 📏 Índices

| Índice             | Tabela       | Campo      | Uso                     |
|--------------------|--------------|------------|-------------------------|
| PRIMARY KEY        | usuario      | id         | Busca por ID            |
| email (UNIQUE)     | usuario      | email      | Login                   |
| documento (UNIQUE) | usuario      | documento  | Validação de duplicação |
| role_id (FK)       | usuario_role | role_id    | Join com role           |
| usuario_id (FK)    | usuario_role | usuario_id | Join com usuario        |

---

## 🎯 Decisões de Design

### Por que não usar coluna "tipo" em usuario?

**Opção rejeitada:**
```sql
CREATE TABLE usuario (
    ...
    tipo ENUM('CLIENTE', 'CONFEITEIRA'),  -- Um só tipo
    ...
);
```

**Problema:** João poderia ser OU cliente OU confeiteira, nunca os dois.

---

### Por que tabela de junção?

**Opção escolhida:**
```
usuario ←→ usuario_role ←→ role
```

**Vantagem:** João pode ter CLIENTE E CONFEITEIRA na mesma conta.

---

### Por que ON DELETE CASCADE em usuario?

Se João é deletado:
```sql
DELETE FROM usuario WHERE id = 1;
-- Automaticamente deleta:
DELETE FROM usuario_role WHERE usuario_id = 1;
```

✅ Mantém dados limpos  
✅ Não fica órfão

---

### Por que ON DELETE RESTRICT em role?

Não permite deletar CLIENTE se houver usuários com essa role:
```sql
DELETE FROM role WHERE id = 1;  -- Vai dar erro se houver usuario_role com role_id=1
```

✅ Protege integridade referencial

---

## 🚀 Próximos Passos

1. **TASK-3, 4, 5** — Criar as migrations SQL baseado nesse DER
2. **TASK-6, 7** — Criar as entidades JPA (Usuario, Role)
3. **TASK-11** — Criar os repositories

Todas as decisões estão documentadas aqui. ✅

---