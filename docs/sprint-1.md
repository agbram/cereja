# Sprint 1 — Base e Infraestrutura

## Objetivo
Configurar o ambiente de desenvolvimento, definir a stack e conectar o banco de dados antes de qualquer código de produto.

## O que foi feito
- Monorepo criado no GitHub com estrutura `backend/`, `frontend/`, `docs/`
- Branches configuradas: `main`, `dev`, `feature/*`
- Backend: Spring Boot criado e conectado ao MySQL
- Frontend: React + Vite rodando com página inicial
- Swagger acessível em `http://localhost:8080/swagger-ui/index.html`
- Usuário dedicado `cereja` criado no MySQL
- Estrutura de pacotes do backend definida
- Documentação inicial criada em `/docs`

## Critérios de sucesso atingidos
- [x] Ambiente rodando localmente
- [x] Banco conectado e funcionando
- [x] Stack documentada e definida
- [x] Código versionado no repositório

## Pendências para Sprint 2
- Modelagem do domínio com o time
- DER (Diagrama Entidade-Relacionamento)
- Entidades JPA
- Migrations com Flyway
- Cadastro e login de usuários