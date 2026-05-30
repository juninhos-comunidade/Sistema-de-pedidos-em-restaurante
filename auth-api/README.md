# Roadmap - Auth API PHP

## Objetivo da API

A Auth API em PHP será responsável por controlar toda a parte de autenticação e usuários do sistema de gerenciamento de restaurante.

Essa API será responsável por:

- Cadastro de usuários
- Login
- Criptografia de senha
- Geração de token JWT
- Validação de credenciais
- Controle de perfis
- Consulta de usuário autenticado
- Gerenciamento de usuários pelo ADMIN

A API Node.js será responsável pelo fluxo interno do restaurante, como mesas, pedidos, itens, produtos e categorias.

A API PHP será responsável apenas por autenticação e usuários.

---

# Relação com a API Node.js

A API Auth PHP deverá gerar um token JWT que será usado pelo frontend para acessar a API Node.js.

Fluxo esperado:

    Usuário faz login na API Auth PHP
      ↓
    API PHP valida email e senha
      ↓
    API PHP gera token JWT
      ↓
    Frontend salva o token
      ↓
    Frontend envia o token nas requisições para a API Node.js
      ↓
    API Node.js valida o token
      ↓
    API Node.js libera ou bloqueia ações conforme o perfil do usuário

Header usado nas requisições:

    Authorization: Bearer token_aqui

---

# Stack Recomendada

- PHP 8+
- Slim Framework ou Laravel
- PostgreSQL
- Docker
- Docker Compose
- JWT
- Dotenv
- Composer
- Bcrypt/password_hash
- PDO ou ORM do framework escolhido

---

# Banco de Dados

A API Auth PHP deverá utilizar PostgreSQL.

A conexão com o banco poderá ser feita via Docker Compose.

Todos os IDs devem ser UUID/GUID.

Isso vale para:

- id
- usuarioId
- perfilId, caso exista tabela separada de perfis
- usuario_id
- perfil_id

Exemplo de UUID/GUID:

    550e8400-e29b-41d4-a716-446655440000

---

# Chave JWT

A chave de assinatura e validação do JWT deve ser um GUID/UUID.

Essa chave deve ficar no arquivo `.env`.

Exemplo:

    JWT_SECRET=8f8d2d2a-42e7-4cf5-9a2d-98ef9d4db0ad

Essa chave será usada para:

- Assinar o token na API Auth PHP
- Validar o token na API Node.js

A API Node.js precisa usar exatamente a mesma chave no `.env`.

Exemplo na API Node.js:

    JWT_SECRET=8f8d2d2a-42e7-4cf5-9a2d-98ef9d4db0ad

Importante:

- Não colocar a chave JWT direto no código.
- Não subir o arquivo `.env` para o GitHub.
- Usar a mesma chave nas duas APIs durante o desenvolvimento.
- A chave precisa ser tratada como segredo do sistema.
- A chave deve ser um GUID/UUID válido.

---

# Exemplo de docker-compose.yml

    services:
      postgres_auth:
        image: postgres:16
        container_name: restaurante-auth-postgres
        restart: always
        environment:
          POSTGRES_DB: restaurante_auth_db
          POSTGRES_USER: auth_user
          POSTGRES_PASSWORD: auth_pass
        ports:
          - "5433:5432"
        volumes:
          - postgres_auth_data:/var/lib/postgresql/data

    volumes:
      postgres_auth_data:

Observação:

A porta externa foi definida como `5433` para evitar conflito com o PostgreSQL da API Node.js, caso as duas APIs rodem localmente ao mesmo tempo.

---

# Exemplo de .env

    APP_PORT=8000
    DB_HOST=localhost
    DB_PORT=5433
    DB_DATABASE=restaurante_auth_db
    DB_USERNAME=auth_user
    DB_PASSWORD=auth_pass
    JWT_SECRET=8f8d2d2a-42e7-4cf5-9a2d-98ef9d4db0ad
    JWT_EXPIRES_IN=86400

Onde:

- `APP_PORT` define a porta da API PHP.
- `DB_HOST` define o host do banco.
- `DB_PORT` define a porta do banco.
- `DB_DATABASE` define o nome do banco.
- `DB_USERNAME` define o usuário do banco.
- `DB_PASSWORD` define a senha do banco.
- `JWT_SECRET` define a chave de assinatura do token.
- `JWT_EXPIRES_IN` define o tempo de expiração do token em segundos.

---

# Perfis do Sistema

A API Auth PHP deverá trabalhar com os seguintes perfis:

    ADMIN
    GARCOM
    RECEPCAO
    COZINHA

---

## ADMIN

O administrador possui acesso completo ao sistema.

Responsabilidades:

- Gerenciar usuários
- Cadastrar usuários
- Editar usuários
- Ativar ou desativar usuários
- Definir perfis
- Acessar todas as funcionalidades da API Node.js

---

## GARCOM

O garçom é responsável pelo atendimento das mesas.

Na API Node.js, poderá:

- Consultar mesas
- Criar pedidos
- Adicionar itens ao pedido
- Atualizar itens do pedido enquanto o pedido estiver aberto
- Cancelar pedido enquanto estiver aberto
- Consultar cardápio

---

## RECEPCAO

A recepção é responsável pelo acompanhamento de mesas e pagamento.

Na API Node.js, poderá:

- Consultar mesas
- Consultar pedidos
- Confirmar pagamento
- Liberar mesa após pagamento

---

## COZINHA

A cozinha é responsável pelo preparo dos pedidos.

Na API Node.js, poderá:

- Consultar pedidos
- Consultar itens dos pedidos
- Alterar pedido para `EM_PREPARO`
- Alterar pedido para `SERVIDO`

---

# Entidades Principais

## Usuario

Representa um usuário do sistema.

Campos:

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| id | UUID/GUID | Identificador único do usuário |
| nome | string | Nome do usuário |
| email | string | Email usado para login |
| senha | string | Senha criptografada |
| perfil | string | Perfil do usuário |
| ativo | boolean | Indica se o usuário pode acessar o sistema |
| dataCriacao | datetime | Data de criação do usuário |

Perfis possíveis:

    ADMIN
    GARCOM
    RECEPCAO
    COZINHA

---

# Regras de Negócio

## Usuário

- Todo usuário deve possuir ID em UUID/GUID.
- Todo usuário deve possuir nome.
- Todo usuário deve possuir email.
- O email deve ser único.
- Todo usuário deve possuir senha.
- A senha deve ser armazenada criptografada.
- A senha nunca deve ser retornada nas respostas da API.
- Todo usuário deve possuir um perfil válido.
- Todo usuário deve possuir status `ativo`.
- Usuários inativos não podem fazer login.
- Apenas ADMIN pode cadastrar usuários.
- Apenas ADMIN pode editar usuários.
- Apenas ADMIN pode desativar usuários.
- Não é recomendado excluir usuários fisicamente.
- O ideal é usar desativação lógica com o campo `ativo`.

---

## Login

- O login deve ser feito com email e senha.
- A API deve validar se o email existe.
- A API deve validar se a senha está correta.
- A API deve validar se o usuário está ativo.
- Caso esteja tudo correto, a API deve gerar um token JWT.
- O token deve conter os dados necessários para a API Node.js autorizar as ações.
- A senha nunca deve ser enviada no token.
- A senha nunca deve aparecer na resposta.

---

## Token JWT

O token JWT deve conter as informações necessárias para identificar o usuário e o perfil.

Payload recomendado:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Miguel",
      "email": "miguel@email.com",
      "perfil": "GARCOM",
      "iat": 1780000000,
      "exp": 1780086400
    }

Campos:

| Campo | Descrição |
| :--- | :--- |
| id | ID do usuário em UUID/GUID |
| nome | Nome do usuário |
| email | Email do usuário |
| perfil | Perfil do usuário |
| iat | Data de criação do token |
| exp | Data de expiração do token |

Importante:

- O campo `id` deve ser UUID/GUID.
- O campo `perfil` deve ser usado pela API Node.js para liberar ou bloquear endpoints.
- O token deve ser assinado usando a chave `JWT_SECRET`.
- A chave `JWT_SECRET` deve ser um GUID/UUID.

---

# Endpoints da API

## Health Check

### Verificar se a API está funcionando

    GET /health

Resposta esperada:

    {
      "status": "Auth API funcionando"
    }

---

# Autenticação

## Login

    POST /auth/login

Body:

    {
      "email": "garcom@restaurante.com",
      "senha": "123456"
    }

Resposta esperada:

    {
      "token": "jwt_gerado_aqui",
      "usuario": {
        "id": "f51c9670-61d7-469d-8016-df40eec632b1",
        "nome": "Garçom Teste",
        "email": "garcom@restaurante.com",
        "perfil": "GARCOM"
      }
    }

Regras:

- Email é obrigatório.
- Senha é obrigatória.
- Email precisa existir.
- Senha precisa estar correta.
- Usuário precisa estar ativo.
- Senha não deve retornar na resposta.
- Token deve ser gerado com JWT.
- Token deve ser assinado usando `JWT_SECRET`.
- `JWT_SECRET` deve ser um GUID/UUID.

---

## Consultar usuário autenticado

    GET /auth/me

Header:

    Authorization: Bearer token_aqui

Resposta esperada:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Garçom Teste",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM",
      "ativo": true
    }

Regras:

- Requisição precisa ter token.
- Token precisa ser válido.
- Usuário precisa existir.
- Senha não deve retornar.

---

## Validar token

    GET /auth/validate

Header:

    Authorization: Bearer token_aqui

Resposta esperada:

    {
      "valido": true,
      "usuario": {
        "id": "f51c9670-61d7-469d-8016-df40eec632b1",
        "nome": "Garçom Teste",
        "email": "garcom@restaurante.com",
        "perfil": "GARCOM"
      }
    }

Regras:

- Endpoint útil caso a API Node.js escolha validar token consultando a API Auth PHP.
- Para o projeto ficar mais simples, a API Node.js pode validar o JWT diretamente usando a mesma `JWT_SECRET`.
- Mesmo assim, esse endpoint é útil para testes.

---

## Logout

    POST /auth/logout

Header:

    Authorization: Bearer token_aqui

Resposta esperada:

    {
      "mensagem": "Logout realizado com sucesso"
    }

Observação:

Com JWT simples, o logout geralmente é feito no frontend apagando o token.

Esse endpoint pode existir apenas para padronizar o fluxo.

Para invalidar token no backend, seria necessário implementar blacklist de tokens, o que pode ser deixado como melhoria futura.

---

# Usuários

## Criar usuário

    POST /usuarios

Perfis permitidos:

    ADMIN

Header:

    Authorization: Bearer token_aqui

Body:

    {
      "nome": "Garçom Teste",
      "email": "garcom@restaurante.com",
      "senha": "123456",
      "perfil": "GARCOM"
    }

Resposta esperada:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Garçom Teste",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM",
      "ativo": true,
      "dataCriacao": "2026-05-29T22:00:00"
    }

Regras:

- Apenas ADMIN pode criar usuário.
- Nome é obrigatório.
- Email é obrigatório.
- Email deve ser único.
- Senha é obrigatória.
- Senha deve ser criptografada antes de salvar.
- Perfil é obrigatório.
- Perfil precisa ser válido.
- Usuário deve iniciar como ativo.
- Senha não deve retornar na resposta.

---

## Listar usuários

    GET /usuarios

Perfis permitidos:

    ADMIN

Header:

    Authorization: Bearer token_aqui

Resposta esperada:

    [
      {
        "id": "f51c9670-61d7-469d-8016-df40eec632b1",
        "nome": "Garçom Teste",
        "email": "garcom@restaurante.com",
        "perfil": "GARCOM",
        "ativo": true,
        "dataCriacao": "2026-05-29T22:00:00"
      },
      {
        "id": "33a9870a-d193-42ad-ae3d-015113fe4b38",
        "nome": "Cozinha Teste",
        "email": "cozinha@restaurante.com",
        "perfil": "COZINHA",
        "ativo": true,
        "dataCriacao": "2026-05-29T22:00:00"
      }
    ]

Regras:

- Apenas ADMIN pode listar usuários.
- Senha não deve retornar na resposta.

---

## Buscar usuário por ID

    GET /usuarios/{id}

Perfis permitidos:

    ADMIN

Header:

    Authorization: Bearer token_aqui

Resposta esperada:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Garçom Teste",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM",
      "ativo": true,
      "dataCriacao": "2026-05-29T22:00:00"
    }

Regras:

- ID deve ser UUID/GUID.
- Apenas ADMIN pode buscar usuário por ID.
- Senha não deve retornar.

---

## Atualizar usuário

    PUT /usuarios/{id}

Perfis permitidos:

    ADMIN

Header:

    Authorization: Bearer token_aqui

Body:

    {
      "nome": "Garçom Atualizado",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM",
      "ativo": true
    }

Resposta esperada:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Garçom Atualizado",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM",
      "ativo": true,
      "dataCriacao": "2026-05-29T22:00:00"
    }

Regras:

- ID deve ser UUID/GUID.
- Apenas ADMIN pode atualizar usuário.
- Nome é obrigatório.
- Email é obrigatório.
- Email não pode repetir.
- Perfil precisa ser válido.
- Senha não deve ser atualizada nesse endpoint.
- Senha não deve retornar na resposta.

---

## Alterar senha do usuário

    PATCH /usuarios/{id}/senha

Perfis permitidos:

    ADMIN

Header:

    Authorization: Bearer token_aqui

Body:

    {
      "novaSenha": "12345678"
    }

Resposta esperada:

    {
      "mensagem": "Senha atualizada com sucesso"
    }

Regras:

- ID deve ser UUID/GUID.
- Apenas ADMIN pode alterar senha de outro usuário.
- Nova senha é obrigatória.
- Nova senha deve ser criptografada antes de salvar.
- Senha nunca deve retornar na resposta.

---

## Alterar minha senha

    PATCH /auth/me/senha

Header:

    Authorization: Bearer token_aqui

Body:

    {
      "senhaAtual": "123456",
      "novaSenha": "12345678"
    }

Resposta esperada:

    {
      "mensagem": "Senha atualizada com sucesso"
    }

Regras:

- Usuário precisa estar autenticado.
- Senha atual é obrigatória.
- Nova senha é obrigatória.
- Senha atual precisa estar correta.
- Nova senha deve ser criptografada antes de salvar.
- Senha nunca deve retornar na resposta.

---

## Desativar usuário

    PATCH /usuarios/{id}/desativar

Perfis permitidos:

    ADMIN

Header:

    Authorization: Bearer token_aqui

Resposta esperada:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Garçom Teste",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM",
      "ativo": false
    }

Regras:

- ID deve ser UUID/GUID.
- Apenas ADMIN pode desativar usuário.
- Usuário desativado não pode fazer login.
- Não excluir usuário fisicamente.

---

## Ativar usuário

    PATCH /usuarios/{id}/ativar

Perfis permitidos:

    ADMIN

Header:

    Authorization: Bearer token_aqui

Resposta esperada:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Garçom Teste",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM",
      "ativo": true
    }

Regras:

- ID deve ser UUID/GUID.
- Apenas ADMIN pode ativar usuário.

---

## Excluir usuário

    DELETE /usuarios/{id}

Perfis permitidos:

    ADMIN

Header:

    Authorization: Bearer token_aqui

Recomendação:

- Evitar exclusão física.
- Preferir desativar usuário com `PATCH /usuarios/{id}/desativar`.

Resposta esperada, caso o grupo decida permitir exclusão:

    {
      "mensagem": "Usuário excluído com sucesso"
    }

---

# Resumo dos Endpoints

## Autenticação

    POST   /auth/login
    GET    /auth/me
    GET    /auth/validate
    POST   /auth/logout
    PATCH  /auth/me/senha

---

## Usuários

    POST   /usuarios
    GET    /usuarios
    GET    /usuarios/{id}
    PUT    /usuarios/{id}
    PATCH  /usuarios/{id}/senha
    PATCH  /usuarios/{id}/desativar
    PATCH  /usuarios/{id}/ativar
    DELETE /usuarios/{id}

---

# Middleware de Autenticação

A API PHP deve possuir um middleware para validar se o usuário está autenticado.

Responsabilidades:

- Verificar se existe header `Authorization`.
- Verificar se o token foi enviado no formato Bearer.
- Validar o token JWT usando `JWT_SECRET`.
- Extrair dados do usuário.
- Disponibilizar o usuário na requisição.
- Bloquear token inválido.
- Bloquear token expirado.

Exemplo lógico:

    Requisição chega
      ↓
    Verifica header Authorization
      ↓
    Verifica token Bearer
      ↓
    Valida assinatura com JWT_SECRET
      ↓
    Valida expiração
      ↓
    Extrai usuário
      ↓
    Permite continuar

---

# Middleware de Permissão

A API PHP deve possuir um middleware para validar perfil do usuário.

Exemplo:

    Endpoint exige ADMIN
      ↓
    Usuário possui perfil GARCOM
      ↓
    Bloqueia acesso

Outro exemplo:

    Endpoint exige ADMIN
      ↓
    Usuário possui perfil ADMIN
      ↓
    Permite acesso

Esse middleware será usado principalmente nos endpoints de gerenciamento de usuários.

---

# Segurança

## Senhas

As senhas devem ser criptografadas usando `password_hash`.

Exemplo lógico:

    password_hash($senha, PASSWORD_BCRYPT)

Para validar a senha:

    password_verify($senhaDigitada, $senhaCriptografada)

Regras:

- Nunca salvar senha pura no banco.
- Nunca retornar senha nas respostas.
- Nunca colocar senha dentro do JWT.
- Nunca logar senha no terminal.
- Nunca enviar senha para a API Node.js.

---

## JWT

Regras:

- A chave `JWT_SECRET` deve ficar no `.env`.
- A chave `JWT_SECRET` deve ser um GUID/UUID.
- A API PHP assina o token.
- A API Node.js valida o token.
- O token deve ter expiração.
- O token deve conter perfil do usuário.
- O token não deve conter senha.
- O token deve ser enviado no header Authorization.

---

# Estrutura Recomendada do Projeto

## Opção com Slim Framework

    src/
    ├── config/
    │   └── database.php
    │
    ├── controllers/
    │   ├── AuthController.php
    │   └── UsuarioController.php
    │
    ├── middlewares/
    │   ├── AuthMiddleware.php
    │   └── RoleMiddleware.php
    │
    ├── repositories/
    │   └── UsuarioRepository.php
    │
    ├── services/
    │   ├── AuthService.php
    │   ├── UsuarioService.php
    │   └── JwtService.php
    │
    ├── routes/
    │   ├── auth.routes.php
    │   └── usuario.routes.php
    │
    ├── enums/
    │   └── PerfilEnum.php
    │
    ├── utils/
    │   └── Response.php
    │
    ├── app.php
    └── server.php

---

## Opção com Laravel

    app/
    ├── Http/
    │   ├── Controllers/
    │   │   ├── AuthController.php
    │   │   └── UsuarioController.php
    │   ├── Middleware/
    │   │   ├── JwtMiddleware.php
    │   │   └── RoleMiddleware.php
    │   └── Requests/
    │       ├── LoginRequest.php
    │       ├── UsuarioRequest.php
    │       └── AlterarSenhaRequest.php
    │
    ├── Models/
    │   └── Usuario.php
    │
    ├── Services/
    │   ├── AuthService.php
    │   ├── UsuarioService.php
    │   └── JwtService.php
    │
    └── Enums/
        └── PerfilEnum.php

    routes/
    └── api.php

    database/
    └── migrations/

---

# Modelagem das Tabelas

## Tabela usuarios

    usuarios
    - id UUID PRIMARY KEY
    - nome VARCHAR NOT NULL
    - email VARCHAR UNIQUE NOT NULL
    - senha VARCHAR NOT NULL
    - perfil VARCHAR NOT NULL
    - ativo BOOLEAN NOT NULL
    - data_criacao TIMESTAMP NOT NULL

---

# Exemplo SQL PostgreSQL com UUID

Para usar UUID automático no PostgreSQL, habilitar a extensão:

    CREATE EXTENSION IF NOT EXISTS "pgcrypto";

Criação da tabela:

    CREATE TABLE usuarios (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      nome VARCHAR(100) NOT NULL,
      email VARCHAR(150) UNIQUE NOT NULL,
      senha VARCHAR(255) NOT NULL,
      perfil VARCHAR(30) NOT NULL,
      ativo BOOLEAN NOT NULL DEFAULT TRUE,
      data_criacao TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
    );

---

# Seed Inicial Recomendado

Para facilitar os testes, criar usuários iniciais para cada perfil.

Usuários sugeridos:

    admin@restaurante.com
    garcom@restaurante.com
    recepcao@restaurante.com
    cozinha@restaurante.com

Senha padrão para testes:

    123456

Perfis:

    ADMIN
    GARCOM
    RECEPCAO
    COZINHA

Importante:

- As senhas do seed também precisam ser salvas criptografadas.
- O seed não deve salvar senha pura no banco.
- A senha `123456` é apenas para ambiente acadêmico/desenvolvimento.

---

# Exemplos de Usuários de Teste

## Admin

    {
      "id": "7bb5378d-6bf4-44dc-8795-6d40255b4ba5",
      "nome": "Admin Teste",
      "email": "admin@restaurante.com",
      "perfil": "ADMIN",
      "ativo": true
    }

---

## Garçom

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Garçom Teste",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM",
      "ativo": true
    }

---

## Recepção

    {
      "id": "2f56f498-33fa-4914-b4b4-c365cc568f87",
      "nome": "Recepção Teste",
      "email": "recepcao@restaurante.com",
      "perfil": "RECEPCAO",
      "ativo": true
    }

---

## Cozinha

    {
      "id": "33a9870a-d193-42ad-ae3d-015113fe4b38",
      "nome": "Cozinha Teste",
      "email": "cozinha@restaurante.com",
      "perfil": "COZINHA",
      "ativo": true
    }

---

# Integração com o Frontend

## Login

O frontend deve enviar email e senha para:

    POST /auth/login

Após o login, o frontend recebe:

    {
      "token": "jwt_gerado_aqui",
      "usuario": {
        "id": "f51c9670-61d7-469d-8016-df40eec632b1",
        "nome": "Garçom Teste",
        "email": "garcom@restaurante.com",
        "perfil": "GARCOM"
      }
    }

O frontend deve armazenar o token e enviar nas requisições para:

- API Auth PHP
- API Node.js

Header:

    Authorization: Bearer token_aqui

---

# Integração com a API Node.js

A API Node.js deverá validar o JWT gerado pela API Auth PHP.

A API Node.js precisa entender o payload:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Garçom Teste",
      "email": "garcom@restaurante.com",
      "perfil": "GARCOM"
    }

Com isso, a API Node.js poderá aplicar permissões como:

- ADMIN pode tudo.
- GARCOM pode criar pedidos e adicionar itens.
- COZINHA pode alterar pedido para preparo e servido.
- RECEPCAO pode confirmar pagamento e liberar mesa.

---

# Validações Obrigatórias

## Login

- [ ] Email obrigatório
- [ ] Senha obrigatória
- [ ] Email precisa existir
- [ ] Senha precisa estar correta
- [ ] Usuário precisa estar ativo
- [ ] Token JWT precisa ser gerado corretamente
- [ ] Token precisa conter ID em UUID/GUID
- [ ] Token precisa conter perfil
- [ ] Token não pode conter senha

---

## Usuário

- [ ] ID em UUID/GUID
- [ ] Nome obrigatório
- [ ] Email obrigatório
- [ ] Email único
- [ ] Senha obrigatória no cadastro
- [ ] Senha criptografada
- [ ] Perfil obrigatório
- [ ] Perfil válido
- [ ] Ativo padrão como true
- [ ] Senha não pode retornar nas respostas

---

## JWT

- [ ] JWT_SECRET no `.env`
- [ ] JWT_SECRET em formato GUID/UUID
- [ ] Token assinado com JWT_SECRET
- [ ] Token com expiração
- [ ] Token com ID do usuário
- [ ] Token com perfil do usuário
- [ ] Token sem senha
- [ ] API Node.js conseguindo validar o token

---

# Prioridade de Desenvolvimento

## Sprint 1 - Configuração Inicial

- [ ] Criar projeto PHP
- [ ] Instalar dependências com Composer
- [ ] Configurar Docker Compose com PostgreSQL
- [ ] Configurar arquivo `.env`
- [ ] Criar conexão com banco de dados
- [ ] Configurar UUID/GUID como padrão para IDs
- [ ] Configurar `JWT_SECRET` como GUID/UUID
- [ ] Criar estrutura inicial de pastas
- [ ] Criar endpoint de health check

---

## Sprint 2 - Banco de Dados e Usuários

- [ ] Criar tabela `usuarios`
- [ ] Criar migration ou script SQL
- [ ] Criar campos com UUID/GUID
- [ ] Criar campo `nome`
- [ ] Criar campo `email`
- [ ] Criar campo `senha`
- [ ] Criar campo `perfil`
- [ ] Criar campo `ativo`
- [ ] Criar campo `data_criacao`
- [ ] Criar seed inicial com ADMIN, GARCOM, RECEPCAO e COZINHA
- [ ] Salvar senhas criptografadas

---

## Sprint 3 - Login e JWT

- [ ] Criar endpoint `POST /auth/login`
- [ ] Validar email obrigatório
- [ ] Validar senha obrigatória
- [ ] Buscar usuário por email
- [ ] Validar senha com `password_verify`
- [ ] Validar se usuário está ativo
- [ ] Gerar token JWT
- [ ] Incluir ID UUID/GUID no token
- [ ] Incluir perfil no token
- [ ] Incluir expiração no token
- [ ] Retornar token e dados públicos do usuário

---

## Sprint 4 - Middleware de Autenticação

- [ ] Criar middleware de autenticação
- [ ] Ler header Authorization
- [ ] Validar formato Bearer
- [ ] Validar token JWT
- [ ] Validar expiração do token
- [ ] Extrair usuário autenticado
- [ ] Bloquear token inválido
- [ ] Bloquear token expirado

---

## Sprint 5 - Middleware de Permissões

- [ ] Criar middleware de perfil
- [ ] Permitir endpoints apenas para ADMIN quando necessário
- [ ] Bloquear usuários sem permissão
- [ ] Retornar erro 403 para perfil sem acesso
- [ ] Testar permissões com todos os perfis

---

## Sprint 6 - Gerenciamento de Usuários

- [ ] Criar endpoint para cadastrar usuário
- [ ] Criar endpoint para listar usuários
- [ ] Criar endpoint para buscar usuário por ID
- [ ] Criar endpoint para atualizar usuário
- [ ] Criar endpoint para alterar senha de usuário
- [ ] Criar endpoint para ativar usuário
- [ ] Criar endpoint para desativar usuário
- [ ] Impedir retorno da senha
- [ ] Validar email único
- [ ] Validar perfil

---

## Sprint 7 - Rotas do Usuário Autenticado

- [ ] Criar endpoint `GET /auth/me`
- [ ] Criar endpoint `PATCH /auth/me/senha`
- [ ] Criar endpoint `GET /auth/validate`
- [ ] Criar endpoint `POST /auth/logout`
- [ ] Validar token nos endpoints protegidos
- [ ] Retornar apenas dados públicos do usuário

---

## Sprint 8 - Integração com API Node.js

- [ ] Confirmar que o payload do JWT possui `id`, `nome`, `email` e `perfil`
- [ ] Confirmar que o `id` do token é UUID/GUID
- [ ] Confirmar que a chave JWT da API PHP é igual à chave JWT da API Node.js
- [ ] Testar token gerado pela API PHP na API Node.js
- [ ] Testar permissões na API Node.js com cada perfil
- [ ] Documentar o payload esperado do token

---

## Sprint 9 - Testes e Documentação

- [ ] Testar login com usuário válido
- [ ] Testar login com senha incorreta
- [ ] Testar login com usuário inexistente
- [ ] Testar login com usuário inativo
- [ ] Testar endpoint protegido sem token
- [ ] Testar endpoint protegido com token inválido
- [ ] Testar endpoint protegido com perfil sem permissão
- [ ] Criar collection Postman ou Insomnia
- [ ] Documentar endpoints
- [ ] Documentar bodies de requisição
- [ ] Documentar exemplos de resposta
- [ ] Documentar uso do Docker Compose
- [ ] Documentar JWT_SECRET como GUID/UUID
- [ ] Criar README completo da API

---

# Códigos de Resposta Recomendados

## 200 OK

Usar quando uma consulta ou ação for realizada com sucesso.

Exemplo:

    Login realizado com sucesso
    Usuário encontrado
    Usuário atualizado

---

## 201 Created

Usar quando um recurso for criado.

Exemplo:

    Usuário criado com sucesso

---

## 400 Bad Request

Usar quando a requisição estiver inválida.

Exemplo:

    Email obrigatório
    Senha obrigatória
    Perfil inválido
    UUID inválido

---

## 401 Unauthorized

Usar quando o usuário não estiver autenticado.

Exemplo:

    Token não enviado
    Token inválido
    Token expirado

---

## 403 Forbidden

Usar quando o usuário está autenticado, mas não possui permissão.

Exemplo:

    Usuário GARCOM tentando cadastrar outro usuário

---

## 404 Not Found

Usar quando o recurso não for encontrado.

Exemplo:

    Usuário não encontrado

---

## 409 Conflict

Usar quando houver conflito de dados.

Exemplo:

    Email já cadastrado

---

# Critérios de Pronto

A API Auth PHP será considerada pronta quando:

- [ ] O projeto estiver rodando localmente.
- [ ] O PostgreSQL estiver configurado via Docker Compose.
- [ ] A API estiver conectando corretamente ao banco PostgreSQL.
- [ ] Todos os IDs estiverem usando UUID/GUID.
- [ ] O `JWT_SECRET` estiver configurado como GUID/UUID.
- [ ] O login estiver funcionando.
- [ ] As senhas estiverem criptografadas.
- [ ] O token JWT estiver sendo gerado corretamente.
- [ ] O token JWT possuir ID, nome, email e perfil.
- [ ] O token JWT não possuir senha.
- [ ] O token JWT possuir expiração.
- [ ] O endpoint `/auth/me` estiver funcionando.
- [ ] O endpoint `/auth/validate` estiver funcionando.
- [ ] O CRUD administrativo de usuários estiver funcionando.
- [ ] Usuários inativos não conseguirem logar.
- [ ] Apenas ADMIN conseguir gerenciar usuários.
- [ ] A API Node.js conseguir validar o token gerado pela API PHP.
- [ ] A collection do Postman ou Insomnia estiver criada.
- [ ] O README estiver documentado.
- [ ] O projeto puder ser iniciado com Docker Compose e comandos simples.

---

# Divisão Recomendada de Responsabilidades

## API Auth PHP

Responsável por:

- Cadastro de usuários
- Login
- Senhas
- Perfis
- Geração de token JWT
- Validação de token
- Gerenciamento de usuários
- Uso de UUID/GUID em todos os IDs
- Uso de `JWT_SECRET` em GUID/UUID

---

## API Node.js

Responsável por:

- Mesas
- Categorias
- Produtos
- Pedidos
- Itens do pedido
- Fluxo interno do restaurante
- Status das mesas
- Status dos pedidos
- Validação do JWT gerado pela API PHP
- Permissões baseadas no perfil recebido no token

---

## Frontend

Responsável por:

- Tela de login
- Envio de email e senha para API Auth PHP
- Armazenamento do token
- Envio do token para API Auth PHP e API Node.js
- Controle de exibição de telas conforme perfil
- Redirecionamento conforme usuário autenticado

---

## QA

Responsável por testar:

- Login
- Token JWT
- Usuário ativo e inativo
- Cadastro de usuários
- Atualização de usuários
- Alteração de senha
- Permissões por perfil
- Integração entre API PHP e API Node.js
- JWT_SECRET compartilhado entre APIs
- IDs em UUID/GUID

---

# Fluxo Final Resumido

    Admin cria usuários na API Auth PHP
      ↓
    Usuário faz login com email e senha
      ↓
    API PHP valida credenciais
      ↓
    API PHP gera JWT com ID, nome, email e perfil
      ↓
    Frontend salva token
      ↓
    Frontend envia token para API Node.js
      ↓
    API Node.js valida token com a mesma JWT_SECRET
      ↓
    API Node.js identifica perfil
      ↓
    API Node.js libera ou bloqueia ações
      ↓
    Usuário acessa as funcionalidades conforme seu perfil
