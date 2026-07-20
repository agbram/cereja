# Sprint 2 — Autenticação e Usuários
## Cereja — Status Geral

**Data de início:** 14/07/2026  
**Objetivo:** Implementar cadastro, login e autenticação com JWT + modelo Role-User  
**Total de Tasks:** 16  
**Status Geral:** 🚧 Em progresso (1/16 concluída)

---

## 📊 Progresso da Sprint

```
████░░░░░░░░░░░░░░░░░░░░░░░░ 6% completo
```

| Categoria         | Tasks | Concluídas | Status |
|-------------------|-------|------------|--------|
| DER e Banco       | 5     | 1          | 20%    |
| Entidades JPA     | 2     | 0          | 0%     |
| Security/JWT      | 3     | 0          | 0%     |
| Transfer Objects  | 1     | 0          | 0%     |
| Repositories      | 1     | 0          | 0%     |
| Services          | 1     | 0          | 0%     |
| Controllers       | 1     | 0          | 0%     |
| Testes            | 1     | 0          | 0%     |
| Documentação      | 1     | 0          | 0%     |

---

## ✅ TASK-1: Criar DER com modelo Role-User

**Status:** ✅ CONCLUÍDA  
**Responsável:** Antonio Gustavo  
**Data de conclusão:** 14/07/2026  
**Estimativa:** 4 horas  
**Tempo real:** 4 horas

---

## 📝 O que foi entregue na TASK-1

### 1. Diagrama Entidade-Relacionamento (DER)

**Arquivo:** `/docs/der.md`

**Conteúdo:**
- ✅ Visualização ASCII do diagrama
- ✅ 3 entidades documentadas: usuario, role, usuario_role
- ✅ Relacionamento Many-to-Many entre usuario e role
- ✅ Cardinalidade explicada (1,n) e (0,n)

**Diagrama visual:**
```
usuario ←(1,n)→ usuario_role ←(0,n)→ role
```

---

### 2. Especificação das Tabelas

**Tabela: usuario**
- id (BIGINT, PK)
- email (VARCHAR 255, UNIQUE)
- senha (VARCHAR 255)
- nome (VARCHAR 255)
- telefone (VARCHAR 20)
- documento (VARCHAR 20, UNIQUE)
- data_criacao (TIMESTAMP)
- ativo (BOOLEAN)

**Tabela: role**
- id (BIGINT, PK)
- nome (VARCHAR 50, UNIQUE)
- Dados padrão: CLIENTE, CONFEITEIRA

**Tabela: usuario_role (junção)**
- usuario_id (FK → usuario.id)
- role_id (FK → role.id)
- Chave primária composta: (usuario_id, role_id)

---

### 3. Relacionamentos Documentados

**Relacionamento 1: Usuario ↔ UsuarioRole**
- Tipo: Um para Muitos (1:N)
- ON DELETE CASCADE (deleta roles ao deletar usuário)
- ON UPDATE CASCADE

**Relacionamento 2: Role ↔ UsuarioRole**
- Tipo: Um para Muitos (1:N)
- ON DELETE RESTRICT (impede deletar role se houver usuários)
- ON UPDATE CASCADE

---

### 4. Decisões de Design Justificadas

**❌ Por que NÃO usar coluna tipo em usuario:**
```sql
-- Opção rejeitada:
CREATE TABLE usuario (
    tipo ENUM('CLIENTE', 'CONFEITEIRA')  -- Um só tipo
);
```
Problema: João não poderia ser cliente E confeiteira na mesma conta.

**✅ Por que SIM usar tabela de junção:**
```sql
-- Opção escolhida:
usuario ↔ usuario_role ↔ role
```
Vantagem: João pode ter CLIENTE + CONFEITEIRA simultaneamente.

---

### 5. Exemplos Práticos

**Cenário 1: João é cliente E confeiteira**
```
usuario: id=1, nome=João, email=joao@email.com
usuario_role: (1, 1) → João é CLIENTE
usuario_role: (1, 2) → João é CONFEITEIRA
```

**Cenário 2: Maria é apenas cliente**
```
usuario: id=2, nome=Maria, email=maria@email.com
usuario_role: (2, 1) → Maria é CLIENTE
```

---

### 6. Queries SQL Documentadas

**Buscar todas as roles de um usuário:**
```sql
SELECT r.nome 
FROM role r
INNER JOIN usuario_role ur ON r.id = ur.role_id
WHERE ur.usuario_id = 1;
```

**Verificar se usuário é CLIENTE:**
```sql
SELECT COUNT(*) > 0 as isCliente
FROM usuario_role ur
INNER JOIN role r ON r.id = ur.role_id
WHERE ur.usuario_id = 1 AND r.nome = 'CLIENTE';
```

---

### 7. Validação de Normalização

✅ **1FN** — Todos os valores são atômicos  
✅ **2FN** — Sem dependências parciais  
✅ **3FN** — Sem dependências transitivas  

Modelo está em **3ª Forma Normal (3NF)** ✅

---

### 8. Documentos Gerados

| Arquivo          | Localização              | Status       |
|------------------|--------------------------|--------------|
| DER              | `/docs/der.md`           | Completo     |
| Modelo Físico    | `/docs/modelo-fisico.md` | Simplificado |
| Backlog Sprint 2 | Notion                   | Atualizado   | 
---

## 🎯 O que vem depois (TASK-2, 3, 4, 5)

### TASK-2: Configurar Flyway (2 horas)
- Adicionar dependência Flyway no Maven
- Configurar paths de migration
- Preparar estrutura para SQL files

### TASK-3: Migration Usuario (1 hora)
- Arquivo: `V1__Create_Usuario_Table.sql`
- Criar tabela usuario com todos os campos
- Adicionar índices para performance

### TASK-4: Migration Role (1 hora)
- Arquivo: `V2__Create_Role_Table.sql`
- Criar tabela role
- Inserir dados padrão (CLIENTE, CONFEITEIRA)

### TASK-5: Migration UsuarioRole (1 hora)
- Arquivo: `V3__Create_UsuarioRole_Table.sql`
- Criar tabela de junção
- Configurar foreign keys

**Ordem de execução:**
```
TASK-1 (DER) ✅
    ↓
TASK-2 (Flyway) → TASK-3 → TASK-4 → TASK-5 (Migrations)
```

---

## 📌 Decisões Levadas para o Code

1. **Modelo User-Role:** Permet múltiplas roles por usuário
2. **Tabela de Junção:** usuario_role conecta muitos-para-muitos
3. **ON DELETE CASCADE:** Limpa automaticamente roles orfãs
4. **ON DELETE RESTRICT:** Protege integrity de roles
5. **Chave Primária Composta:** (usuario_id, role_id) em usuario_role

---

## 🔐 Notas de Segurança

- [ ] Senha deve ser hasheada com BCrypt (será em TASK-16)
- [ ] Email deve ser único (FK uniqueness criado)
- [ ] Documento deve ser único (FK uniqueness criado)
- [ ] Roles serão validadas no backend (TASK-12)

---

## ✨ Checklist de Validação TASK-1

- [x] DER criado com 3 entidades
- [x] Tabelas especificadas completamente
- [x] Relacionamentos documentados
- [x] Cardinalidade definida
- [x] Foreign keys configuradas
- [x] Decisões de design justificadas
- [x] Exemplos práticos inclusos
- [x] Queries SQL de exemplo
- [x] Normalização validada
- [x] Arquivo `/docs/der.md` criado
- [x] Modelo pronto para migrations

**Status:** ✅ PRONTO PARA PRÓXIMAS TASKS

---

## 📚 Documentação Complementar

Documentos criados na Sprint 2:
1. `/docs/der.md` — Especificação completa do banco
2. `/docs/sprint-2.md` — Este arquivo (status da sprint)
3. `Sprint 2 Backlog` (Notion) — Tasks com responsáveis

---

## 🚀 Próximos Passos Imediatos

1. ✅ TASK-1 concluída
2. 👉 **TASK-2** — Flyway setup (Começar HOJE)
3. **TASK-3, 4, 5** — Criar migrations (paralelo)
4. **TASK-6, 7** — Entities JPA (após migrations)

---

**Sprint 2 iniciada!** 🍒  
Próxima atualização após TASK-2. ✅