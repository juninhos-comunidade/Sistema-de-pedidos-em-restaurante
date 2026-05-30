# Roadmap - Restaurante API Node.js

## Objetivo da API

A Restaurante API em Node.js será responsável por controlar todo o fluxo interno do restaurante, incluindo:

- Mesas
- Categorias do cardápio
- Produtos do cardápio
- Pedidos
- Itens dos pedidos
- Fluxo de status dos pedidos
- Fluxo de status das mesas

A autenticação de usuários será responsabilidade da API Auth em PHP.

A API Node.js não fará login nem cadastro de usuários. Ela apenas deverá receber e validar o token enviado pela API Auth PHP, identificando o perfil do usuário autenticado.

Perfis esperados:

- ADMIN
- GARCOM
- RECEPCAO
- COZINHA

A conexão com o banco de dados deverá ser feita por meio de Docker Compose utilizando PostgreSQL.

Todos os identificadores da API devem ser UUID/GUID.

---

# Stack Recomendada

- Node.js
- Express.js
- PostgreSQL
- Docker
- Docker Compose
- Prisma ORM ou Sequelize
- JWT
- Bcrypt somente na API Auth PHP
- Dotenv
- Cors
- Nodemon em ambiente de desenvolvimento

---

# Banco de Dados

A API Node.js deverá utilizar um banco PostgreSQL rodando via Docker Compose.

Todos os campos de identificação devem ser UUID/GUID.

Isso vale para:

- id
- mesaId
- categoriaId
- produtoId
- pedidoId
- itemPedidoId
- mesa_id
- categoria_id
- produto_id
- pedido_id

Exemplo de UUID:

    550e8400-e29b-41d4-a716-446655440000

## Exemplo de docker-compose.yml

    services:
      postgres:
        image: postgres:16
        container_name: restaurante-postgres
        restart: always
        environment:
          POSTGRES_DB: restaurante_db
          POSTGRES_USER: restaurante_user
          POSTGRES_PASSWORD: restaurante_pass
        ports:
          - "5432:5432"
        volumes:
          - postgres_data:/var/lib/postgresql/data

    volumes:
      postgres_data:

## Exemplo de .env

    PORT=3000
    DATABASE_URL=postgresql://restaurante_user:restaurante_pass@localhost:5432/restaurante_db
    JWT_SECRET=chave_usada_tambem_pela_api_auth_php

Observação:

A variável `JWT_SECRET` precisa ser compatível com a chave usada pela API Auth PHP, caso a API Node.js valide o token diretamente.

Outra possibilidade é a API Node.js consultar a API Auth PHP para validar o token, mas para o projeto ficar mais simples, o ideal é usar JWT compartilhado.

---

# Entidades Principais

## Mesa

Representa uma mesa física do restaurante.

Campos:

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| id | UUID/GUID | Identificador único da mesa |
| numero | number | Número da mesa |
| status | string | Status atual da mesa |

Status possíveis:

    DISPONIVEL
    OCUPADA
    AGUARDANDO_PAGAMENTO
    FINALIZADA

---

## Categoria

Representa uma categoria do cardápio.

Exemplos:

- Bebidas
- Entradas
- Pratos principais
- Sobremesas

Campos:

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| id | UUID/GUID | Identificador único da categoria |
| nome | string | Nome da categoria |

---

## Produto

Representa um produto disponível no cardápio.

Campos:

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| id | UUID/GUID | Identificador único do produto |
| nome | string | Nome do produto |
| descricao | string | Descrição do produto |
| preco | decimal | Preço do produto |
| categoriaId | UUID/GUID | Categoria vinculada ao produto |

---

## Pedido

Representa um pedido feito para uma mesa.

Campos:

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| id | UUID/GUID | Identificador único do pedido |
| mesaId | UUID/GUID | Mesa vinculada ao pedido |
| status | string | Status atual do pedido |
| valorTotal | decimal | Valor total do pedido |
| dataCriacao | datetime | Data de criação do pedido |

Status possíveis:

    ABERTO
    EM_PREPARO
    SERVIDO
    PAGO
    CANCELADO

---

## ItemPedido

Representa um produto dentro de um pedido.

Campos:

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| id | UUID/GUID | Identificador único do item |
| pedidoId | UUID/GUID | Pedido vinculado |
| produtoId | UUID/GUID | Produto selecionado |
| quantidade | number | Quantidade do produto |
| valorUnitario | decimal | Valor do produto no momento da venda |

---

# Fluxo de Status dos Pedidos

## Fluxo Principal

    ABERTO
      ↓
    EM_PREPARO
      ↓
    SERVIDO
      ↓
    PAGO

## Fluxo de Cancelamento

    ABERTO
      ↓
    CANCELADO

---

# Regras de Negócio

## Regras de Mesa

- Toda mesa deve iniciar com status `DISPONIVEL`.
- Uma mesa não pode ter número repetido.
- Ao abrir um pedido para uma mesa, o status da mesa deve mudar para `OCUPADA`.
- Não deve ser possível abrir um novo pedido para uma mesa com status `OCUPADA`.
- Quando o pedido for servido, a mesa deve mudar para `AGUARDANDO_PAGAMENTO`.
- Quando o pedido for pago, a mesa pode mudar para `FINALIZADA`.
- Após a finalização, a mesa pode ser liberada e voltar para `DISPONIVEL`.
- Não deve ser possível excluir uma mesa que possui pedido em aberto, em preparo ou servido.

---

## Regras de Categoria

- Uma categoria deve possuir nome obrigatório.
- O nome da categoria não deve ser vazio.
- O ideal é não permitir categorias duplicadas com o mesmo nome.
- Não deve ser possível excluir uma categoria que possui produtos vinculados.

---

## Regras de Produto

- Um produto deve possuir nome obrigatório.
- Um produto deve possuir descrição.
- Um produto deve possuir preço maior que zero.
- Um produto deve obrigatoriamente pertencer a uma categoria.
- Não deve ser possível cadastrar produto vinculado a uma categoria inexistente.
- Caso um produto já tenha sido usado em pedidos, o ideal é evitar exclusão física.
- Para produtos já utilizados, recomenda-se exclusão lógica futuramente, usando um campo `ativo`.

---

## Regras de Pedido

- Todo pedido deve estar vinculado a uma mesa.
- Um pedido sempre deve iniciar com status `ABERTO`.
- Um pedido só pode ser criado para uma mesa `DISPONIVEL`.
- Ao criar um pedido, a mesa deve mudar para `OCUPADA`.
- Um pedido só pode ir para `EM_PREPARO` se estiver `ABERTO`.
- Um pedido só pode ir para `SERVIDO` se estiver `EM_PREPARO`.
- Um pedido só pode ir para `PAGO` se estiver `SERVIDO`.
- Um pedido só pode ser cancelado se estiver `ABERTO`.
- Um pedido cancelado não pode receber novos itens.
- Um pedido pago não pode ser alterado.
- O valor total do pedido deve ser calculado automaticamente com base nos itens.
- O valor total não deve ser informado manualmente pelo usuário.

---

## Regras de ItemPedido

- Um item só pode ser adicionado a um pedido com status `ABERTO`.
- O produto informado precisa existir.
- A quantidade precisa ser maior que zero.
- O `valorUnitario` deve ser copiado do preço atual do produto no momento da venda.
- O valor do produto não deve ser buscado novamente depois para pedidos antigos.
- Sempre que um item for adicionado, atualizado ou removido, o `valorTotal` do pedido deve ser recalculado.
- Não deve ser possível alterar itens de pedidos em preparo, servidos, pagos ou cancelados.

---

# Perfis e Permissões

## ADMIN

O administrador pode acessar todos os recursos da API.

Permissões:

- Cadastrar mesas
- Listar mesas
- Buscar mesa por ID
- Atualizar mesas
- Excluir mesas
- Cadastrar categorias
- Listar categorias
- Buscar categoria por ID
- Atualizar categorias
- Excluir categorias
- Cadastrar produtos
- Listar produtos
- Buscar produto por ID
- Atualizar produtos
- Excluir produtos
- Criar pedidos
- Listar pedidos
- Buscar pedido por ID
- Adicionar itens ao pedido
- Atualizar itens do pedido
- Remover itens do pedido
- Cancelar pedidos
- Alterar status dos pedidos
- Confirmar pagamento
- Liberar mesas

---

## GARCOM

O garçom é responsável pelo atendimento das mesas e criação de pedidos.

Permissões:

- Listar mesas
- Buscar mesa por ID
- Consultar categorias
- Consultar produtos
- Criar pedido para mesa disponível
- Adicionar itens ao pedido
- Atualizar quantidade de itens enquanto o pedido estiver aberto
- Remover itens enquanto o pedido estiver aberto
- Consultar pedidos
- Buscar pedido por ID
- Cancelar pedido enquanto estiver aberto

Não pode:

- Cadastrar produtos
- Editar produtos
- Excluir produtos
- Cadastrar categorias
- Editar categorias
- Excluir categorias
- Marcar pedido como pago
- Gerenciar mesas administrativamente

---

## COZINHA

A cozinha é responsável por acompanhar e atualizar o preparo dos pedidos.

Permissões:

- Listar pedidos
- Buscar pedido por ID
- Listar pedidos em aberto
- Listar pedidos em preparo
- Alterar pedido de `ABERTO` para `EM_PREPARO`
- Alterar pedido de `EM_PREPARO` para `SERVIDO`
- Consultar itens do pedido
- Consultar produtos

Não pode:

- Criar pedidos
- Adicionar itens
- Remover itens
- Cancelar pedidos
- Confirmar pagamento
- Gerenciar mesas
- Gerenciar cardápio

---

## RECEPCAO

A recepção é responsável por acompanhar mesas e pagamento.

Permissões:

- Listar mesas
- Buscar mesa por ID
- Listar pedidos
- Buscar pedido por ID
- Visualizar pedidos servidos
- Marcar pedido como `PAGO`
- Liberar mesa após pagamento

Não pode:

- Criar produtos
- Editar produtos
- Excluir produtos
- Criar categorias
- Editar categorias
- Excluir categorias
- Adicionar itens ao pedido
- Remover itens do pedido
- Alterar pedido para preparo
- Alterar pedido para servido

---

# Endpoints da API

## Mesas

### Criar mesa

    POST /mesas

Perfis permitidos:

    ADMIN

Body:

    {
      "numero": 1
    }

Resposta esperada:

    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "numero": 1,
      "status": "DISPONIVEL"
    }

Regras:

- Número da mesa é obrigatório.
- Número da mesa deve ser único.
- Mesa deve iniciar como `DISPONIVEL`.

---

### Listar mesas

    GET /mesas

Perfis permitidos:

    ADMIN
    GARCOM
    RECEPCAO

Resposta esperada:

    [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "numero": 1,
        "status": "DISPONIVEL"
      },
      {
        "id": "b1bb9c72-3e5c-4e97-8f16-18d630d6e4d7",
        "numero": 2,
        "status": "OCUPADA"
      }
    ]

---

### Buscar mesa por ID

    GET /mesas/{id}

Perfis permitidos:

    ADMIN
    GARCOM
    RECEPCAO

Resposta esperada:

    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "numero": 1,
      "status": "DISPONIVEL"
    }

---

### Atualizar mesa

    PUT /mesas/{id}

Perfis permitidos:

    ADMIN

Body:

    {
      "numero": 2,
      "status": "DISPONIVEL"
    }

Regras:

- Número não pode repetir.
- Status precisa ser válido.

---

### Excluir mesa

    DELETE /mesas/{id}

Perfis permitidos:

    ADMIN

Regras:

- Não excluir mesa com pedido aberto.
- Não excluir mesa com pedido em preparo.
- Não excluir mesa com pedido servido.
- Não excluir mesa aguardando pagamento.

---

### Liberar mesa

    PATCH /mesas/{id}/liberar

Perfis permitidos:

    ADMIN
    RECEPCAO

Regras:

- Mesa só pode ser liberada após o pedido estar `PAGO`.
- Status da mesa deve voltar para `DISPONIVEL`.

---

# Categorias

## Criar categoria

    POST /categorias

Perfis permitidos:

    ADMIN

Body:

    {
      "nome": "Bebidas"
    }

Resposta esperada:

    {
      "id": "4cd1d7a1-74a8-41e2-9876-b845cb9dbd91",
      "nome": "Bebidas"
    }

---

## Listar categorias

    GET /categorias

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

Resposta esperada:

    [
      {
        "id": "4cd1d7a1-74a8-41e2-9876-b845cb9dbd91",
        "nome": "Bebidas"
      },
      {
        "id": "ed2e148e-45ff-4721-b576-44e8fc14297f",
        "nome": "Pratos principais"
      }
    ]

---

## Buscar categoria por ID

    GET /categorias/{id}

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

Resposta esperada:

    {
      "id": "4cd1d7a1-74a8-41e2-9876-b845cb9dbd91",
      "nome": "Bebidas"
    }

---

## Atualizar categoria

    PUT /categorias/{id}

Perfis permitidos:

    ADMIN

Body:

    {
      "nome": "Bebidas Geladas"
    }

---

## Excluir categoria

    DELETE /categorias/{id}

Perfis permitidos:

    ADMIN

Regras:

- Não permitir excluir categoria com produtos vinculados.

---

# Produtos

## Criar produto

    POST /produtos

Perfis permitidos:

    ADMIN

Body:

    {
      "nome": "Coca-Cola",
      "descricao": "Refrigerante lata 350ml",
      "preco": 6.5,
      "categoriaId": "4cd1d7a1-74a8-41e2-9876-b845cb9dbd91"
    }

Resposta esperada:

    {
      "id": "26f5c47f-40ef-46fd-8c70-6ec85a8f43b7",
      "nome": "Coca-Cola",
      "descricao": "Refrigerante lata 350ml",
      "preco": 6.5,
      "categoriaId": "4cd1d7a1-74a8-41e2-9876-b845cb9dbd91"
    }

Regras:

- Nome é obrigatório.
- Descrição é obrigatória.
- Preço deve ser maior que zero.
- Categoria precisa existir.

---

## Listar produtos

    GET /produtos

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

Resposta esperada:

    [
      {
        "id": "26f5c47f-40ef-46fd-8c70-6ec85a8f43b7",
        "nome": "Coca-Cola",
        "descricao": "Refrigerante lata 350ml",
        "preco": 6.5,
        "categoriaId": "4cd1d7a1-74a8-41e2-9876-b845cb9dbd91"
      }
    ]

---

## Listar produtos por categoria

    GET /produtos?categoriaId=4cd1d7a1-74a8-41e2-9876-b845cb9dbd91

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

---

## Buscar produto por ID

    GET /produtos/{id}

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

Resposta esperada:

    {
      "id": "26f5c47f-40ef-46fd-8c70-6ec85a8f43b7",
      "nome": "Coca-Cola",
      "descricao": "Refrigerante lata 350ml",
      "preco": 6.5,
      "categoriaId": "4cd1d7a1-74a8-41e2-9876-b845cb9dbd91"
    }

---

## Atualizar produto

    PUT /produtos/{id}

Perfis permitidos:

    ADMIN

Body:

    {
      "nome": "Coca-Cola",
      "descricao": "Refrigerante lata 350ml",
      "preco": 7,
      "categoriaId": "4cd1d7a1-74a8-41e2-9876-b845cb9dbd91"
    }

Regras:

- Produto precisa existir.
- Categoria precisa existir.
- Preço precisa ser maior que zero.

---

## Excluir produto

    DELETE /produtos/{id}

Perfis permitidos:

    ADMIN

Regras:

- Produto não deve ser excluído fisicamente se já estiver vinculado a algum pedido.
- Caso o grupo queira simplificar, pode permitir exclusão apenas se o produto nunca foi usado.

---

# Pedidos

## Criar pedido

    POST /pedidos

Perfis permitidos:

    ADMIN
    GARCOM

Body:

    {
      "mesaId": "550e8400-e29b-41d4-a716-446655440000"
    }

Resposta esperada:

    {
      "id": "a13b6a8c-51ef-40f7-9183-90f1c731e2ab",
      "mesaId": "550e8400-e29b-41d4-a716-446655440000",
      "status": "ABERTO",
      "valorTotal": 0,
      "dataCriacao": "2026-05-29T22:00:00"
    }

Regras:

- Mesa precisa existir.
- Mesa precisa estar `DISPONIVEL`.
- Pedido deve iniciar como `ABERTO`.
- Valor total deve iniciar como `0`.
- Data de criação deve ser gerada automaticamente.
- Mesa deve mudar para `OCUPADA`.

---

## Listar pedidos

    GET /pedidos

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

Resposta esperada:

    [
      {
        "id": "a13b6a8c-51ef-40f7-9183-90f1c731e2ab",
        "mesaId": "550e8400-e29b-41d4-a716-446655440000",
        "status": "ABERTO",
        "valorTotal": 38.5,
        "dataCriacao": "2026-05-29T22:00:00"
      }
    ]

---

## Listar pedidos por status

    GET /pedidos?status=ABERTO

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

Exemplos:

    GET /pedidos?status=ABERTO
    GET /pedidos?status=EM_PREPARO
    GET /pedidos?status=SERVIDO
    GET /pedidos?status=PAGO
    GET /pedidos?status=CANCELADO

---

## Buscar pedido por ID

    GET /pedidos/{id}

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

Resposta recomendada:

    {
      "id": "a13b6a8c-51ef-40f7-9183-90f1c731e2ab",
      "mesaId": "550e8400-e29b-41d4-a716-446655440000",
      "numeroMesa": 5,
      "status": "ABERTO",
      "valorTotal": 38.5,
      "dataCriacao": "2026-05-29T22:00:00",
      "itens": [
        {
          "id": "0d14d9ad-2b15-439c-9a58-f4f4e6ec7257",
          "produtoId": "26f5c47f-40ef-46fd-8c70-6ec85a8f43b7",
          "nomeProduto": "X-Burger",
          "quantidade": 2,
          "valorUnitario": 15,
          "subtotal": 30
        },
        {
          "id": "03d10a2a-3b98-4b1f-b6b8-4d5f3581ac43",
          "produtoId": "3390a2af-36a9-4645-9f4d-149fe7f72f32",
          "nomeProduto": "Coca-Cola",
          "quantidade": 1,
          "valorUnitario": 8.5,
          "subtotal": 8.5
        }
      ]
    }

---

## Cancelar pedido

    PATCH /pedidos/{id}/cancelar

Perfis permitidos:

    ADMIN
    GARCOM

Regras:

- Pedido precisa existir.
- Pedido precisa estar `ABERTO`.
- Pedido muda para `CANCELADO`.
- Mesa pode voltar para `DISPONIVEL`.

Fluxo:

    ABERTO -> CANCELADO

---

## Enviar pedido para preparo

    PATCH /pedidos/{id}/preparar

Perfis permitidos:

    ADMIN
    COZINHA

Regras:

- Pedido precisa existir.
- Pedido precisa estar `ABERTO`.
- Pedido precisa possuir pelo menos um item.
- Pedido muda para `EM_PREPARO`.

Fluxo:

    ABERTO -> EM_PREPARO

---

## Marcar pedido como servido

    PATCH /pedidos/{id}/servir

Perfis permitidos:

    ADMIN
    COZINHA

Regras:

- Pedido precisa existir.
- Pedido precisa estar `EM_PREPARO`.
- Pedido muda para `SERVIDO`.
- Mesa muda para `AGUARDANDO_PAGAMENTO`.

Fluxo:

    EM_PREPARO -> SERVIDO

---

## Marcar pedido como pago

    PATCH /pedidos/{id}/pagar

Perfis permitidos:

    ADMIN
    RECEPCAO

Regras:

- Pedido precisa existir.
- Pedido precisa estar `SERVIDO`.
- Pedido muda para `PAGO`.
- Mesa muda para `FINALIZADA`.

Fluxo:

    SERVIDO -> PAGO

---

## Excluir pedido

    DELETE /pedidos/{id}

Perfis permitidos:

    ADMIN

Regras:

- Preferencialmente não excluir pedidos pagos.
- Para histórico, o ideal é manter o pedido salvo.
- Cancelamento deve ser feito pelo endpoint de cancelar pedido.

---

# Itens do Pedido

## Adicionar item ao pedido

    POST /pedidos/{pedidoId}/itens

Perfis permitidos:

    ADMIN
    GARCOM

Body:

    {
      "produtoId": "26f5c47f-40ef-46fd-8c70-6ec85a8f43b7",
      "quantidade": 2
    }

Resposta esperada:

    {
      "id": "0d14d9ad-2b15-439c-9a58-f4f4e6ec7257",
      "pedidoId": "a13b6a8c-51ef-40f7-9183-90f1c731e2ab",
      "produtoId": "26f5c47f-40ef-46fd-8c70-6ec85a8f43b7",
      "quantidade": 2,
      "valorUnitario": 6.5
    }

Regras:

- Pedido precisa existir.
- Pedido precisa estar `ABERTO`.
- Produto precisa existir.
- Quantidade precisa ser maior que zero.
- Valor unitário deve ser copiado do preço atual do produto.
- Valor total do pedido deve ser recalculado.

---

## Listar itens do pedido

    GET /pedidos/{pedidoId}/itens

Perfis permitidos:

    ADMIN
    GARCOM
    COZINHA
    RECEPCAO

Resposta esperada:

    [
      {
        "id": "0d14d9ad-2b15-439c-9a58-f4f4e6ec7257",
        "pedidoId": "a13b6a8c-51ef-40f7-9183-90f1c731e2ab",
        "produtoId": "26f5c47f-40ef-46fd-8c70-6ec85a8f43b7",
        "nomeProduto": "Coca-Cola",
        "quantidade": 2,
        "valorUnitario": 6.5,
        "subtotal": 13
      }
    ]

---

## Atualizar quantidade de item

    PUT /pedidos/{pedidoId}/itens/{itemId}

Perfis permitidos:

    ADMIN
    GARCOM

Body:

    {
      "quantidade": 3
    }

Regras:

- Pedido precisa estar `ABERTO`.
- Item precisa pertencer ao pedido informado.
- Quantidade precisa ser maior que zero.
- Valor total do pedido deve ser recalculado.

---

## Remover item do pedido

    DELETE /pedidos/{pedidoId}/itens/{itemId}

Perfis permitidos:

    ADMIN
    GARCOM

Regras:

- Pedido precisa estar `ABERTO`.
- Item precisa pertencer ao pedido informado.
- Valor total do pedido deve ser recalculado.

---

# Integração com API Auth PHP

A API Auth PHP será responsável por:

- Login
- Cadastro de usuários
- Controle de senha
- Geração de token JWT
- Perfis dos usuários

A API Node.js será responsável por:

- Receber o token no header da requisição
- Validar o token
- Identificar o usuário autenticado
- Identificar o perfil do usuário
- Permitir ou bloquear acesso aos endpoints

Header esperado:

    Authorization: Bearer token_aqui

Payload esperado no token:

    {
      "id": "f51c9670-61d7-469d-8016-df40eec632b1",
      "nome": "Miguel",
      "email": "miguel@email.com",
      "perfil": "GARCOM"
    }

Perfis aceitos pela API Node.js:

    ADMIN
    GARCOM
    RECEPCAO
    COZINHA

---

# Middleware de Autenticação

A API Node.js deve possuir um middleware para validar se o usuário está autenticado.

Responsabilidades:

- Verificar se existe header `Authorization`.
- Verificar se o token foi enviado no formato Bearer.
- Validar o token JWT.
- Extrair dados do usuário.
- Disponibilizar o usuário na requisição.

Exemplo lógico:

    Requisição chega
      ↓
    Verifica Authorization
      ↓
    Valida token JWT
      ↓
    Extrai usuário e perfil
      ↓
    Permite continuar

---

# Middleware de Permissão

A API Node.js deve possuir um middleware para validar o perfil do usuário.

Exemplo lógico:

    Endpoint exige ADMIN
      ↓
    Usuário possui perfil GARCOM
      ↓
    Bloqueia acesso

Outro exemplo:

    Endpoint permite ADMIN ou GARCOM
      ↓
    Usuário possui perfil GARCOM
      ↓
    Permite acesso

---

# Estrutura Recomendada do Projeto

    src/
    ├── config/
    │   └── database.js
    │
    ├── controllers/
    │   ├── mesa.controller.js
    │   ├── categoria.controller.js
    │   ├── produto.controller.js
    │   ├── pedido.controller.js
    │   └── itemPedido.controller.js
    │
    ├── services/
    │   ├── mesa.service.js
    │   ├── categoria.service.js
    │   ├── produto.service.js
    │   ├── pedido.service.js
    │   └── itemPedido.service.js
    │
    ├── repositories/
    │   ├── mesa.repository.js
    │   ├── categoria.repository.js
    │   ├── produto.repository.js
    │   ├── pedido.repository.js
    │   └── itemPedido.repository.js
    │
    ├── routes/
    │   ├── mesa.routes.js
    │   ├── categoria.routes.js
    │   ├── produto.routes.js
    │   ├── pedido.routes.js
    │   └── itemPedido.routes.js
    │
    ├── middlewares/
    │   ├── auth.middleware.js
    │   ├── role.middleware.js
    │   └── error.middleware.js
    │
    ├── enums/
    │   ├── mesaStatus.enum.js
    │   └── pedidoStatus.enum.js
    │
    ├── utils/
    │   └── calcularValorTotal.js
    │
    ├── app.js
    └── server.js

---

# Modelagem das Tabelas

## Tabela mesas

    mesas
    - id UUID PRIMARY KEY
    - numero INTEGER UNIQUE NOT NULL
    - status VARCHAR NOT NULL

---

## Tabela categorias

    categorias
    - id UUID PRIMARY KEY
    - nome VARCHAR NOT NULL

---

## Tabela produtos

    produtos
    - id UUID PRIMARY KEY
    - nome VARCHAR NOT NULL
    - descricao TEXT NOT NULL
    - preco DECIMAL(10,2) NOT NULL
    - categoria_id UUID NOT NULL
    - FOREIGN KEY (categoria_id) REFERENCES categorias(id)

---

## Tabela pedidos

    pedidos
    - id UUID PRIMARY KEY
    - mesa_id UUID NOT NULL
    - status VARCHAR NOT NULL
    - valor_total DECIMAL(10,2) NOT NULL
    - data_criacao TIMESTAMP NOT NULL
    - FOREIGN KEY (mesa_id) REFERENCES mesas(id)

---

## Tabela itens_pedido

    itens_pedido
    - id UUID PRIMARY KEY
    - pedido_id UUID NOT NULL
    - produto_id UUID NOT NULL
    - quantidade INTEGER NOT NULL
    - valor_unitario DECIMAL(10,2) NOT NULL
    - FOREIGN KEY (pedido_id) REFERENCES pedidos(id)
    - FOREIGN KEY (produto_id) REFERENCES produtos(id)

---

# Exemplo SQL PostgreSQL com UUID

Para usar UUID automático no PostgreSQL, habilitar a extensão:

    CREATE EXTENSION IF NOT EXISTS "pgcrypto";

Exemplo de criação das tabelas:

    CREATE TABLE mesas (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      numero INTEGER UNIQUE NOT NULL,
      status VARCHAR(30) NOT NULL
    );

    CREATE TABLE categorias (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      nome VARCHAR(100) NOT NULL
    );

    CREATE TABLE produtos (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      nome VARCHAR(100) NOT NULL,
      descricao TEXT NOT NULL,
      preco DECIMAL(10,2) NOT NULL,
      categoria_id UUID NOT NULL,
      CONSTRAINT fk_produtos_categorias
        FOREIGN KEY (categoria_id)
        REFERENCES categorias(id)
    );

    CREATE TABLE pedidos (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      mesa_id UUID NOT NULL,
      status VARCHAR(30) NOT NULL,
      valor_total DECIMAL(10,2) NOT NULL DEFAULT 0,
      data_criacao TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
      CONSTRAINT fk_pedidos_mesas
        FOREIGN KEY (mesa_id)
        REFERENCES mesas(id)
    );

    CREATE TABLE itens_pedido (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      pedido_id UUID NOT NULL,
      produto_id UUID NOT NULL,
      quantidade INTEGER NOT NULL,
      valor_unitario DECIMAL(10,2) NOT NULL,
      CONSTRAINT fk_itens_pedido_pedidos
        FOREIGN KEY (pedido_id)
        REFERENCES pedidos(id),
      CONSTRAINT fk_itens_pedido_produtos
        FOREIGN KEY (produto_id)
        REFERENCES produtos(id)
    );

---

# Exemplo Prisma com UUID

    model Mesa {
      id      String   @id @default(uuid()) @db.Uuid
      numero  Int      @unique
      status  String
      pedidos Pedido[]
    }

    model Categoria {
      id       String    @id @default(uuid()) @db.Uuid
      nome     String
      produtos Produto[]
    }

    model Produto {
      id          String       @id @default(uuid()) @db.Uuid
      nome        String
      descricao   String
      preco       Decimal
      categoriaId String       @db.Uuid
      categoria   Categoria    @relation(fields: [categoriaId], references: [id])
      itens       ItemPedido[]
    }

    model Pedido {
      id          String       @id @default(uuid()) @db.Uuid
      mesaId      String       @db.Uuid
      status      String
      valorTotal  Decimal      @default(0)
      dataCriacao DateTime     @default(now())
      mesa        Mesa         @relation(fields: [mesaId], references: [id])
      itens       ItemPedido[]
    }

    model ItemPedido {
      id            String   @id @default(uuid()) @db.Uuid
      pedidoId      String   @db.Uuid
      produtoId     String   @db.Uuid
      quantidade    Int
      valorUnitario Decimal
      pedido        Pedido   @relation(fields: [pedidoId], references: [id])
      produto       Produto  @relation(fields: [produtoId], references: [id])
    }

---

# Relacionamentos

## Mesa e Pedido

Uma mesa pode ter vários pedidos ao longo do tempo.

    Mesa 1:N Pedido

Um pedido pertence a uma mesa.

O relacionamento deve ser feito por UUID/GUID:

    pedidos.mesa_id -> mesas.id

---

## Categoria e Produto

Uma categoria pode ter vários produtos.

    Categoria 1:N Produto

Um produto pertence a uma categoria.

O relacionamento deve ser feito por UUID/GUID:

    produtos.categoria_id -> categorias.id

---

## Pedido e ItemPedido

Um pedido pode ter vários itens.

    Pedido 1:N ItemPedido

Um item pertence a um pedido.

O relacionamento deve ser feito por UUID/GUID:

    itens_pedido.pedido_id -> pedidos.id

---

## Produto e ItemPedido

Um produto pode aparecer em vários itens de pedido.

    Produto 1:N ItemPedido

Um item pertence a um produto.

O relacionamento deve ser feito por UUID/GUID:

    itens_pedido.produto_id -> produtos.id

---

# Fluxo Geral do Sistema

## 1. Admin cadastra as mesas

Endpoint:

    POST /mesas

Resultado:

    Mesa criada com status DISPONIVEL

---

## 2. Admin cadastra categorias

Endpoint:

    POST /categorias

Exemplos:

    Bebidas
    Entradas
    Pratos principais
    Sobremesas

---

## 3. Admin cadastra produtos

Endpoint:

    POST /produtos

Cada produto deve estar vinculado a uma categoria usando `categoriaId` em UUID/GUID.

---

## 4. Garçom abre pedido para uma mesa

Endpoint:

    POST /pedidos

Body:

    {
      "mesaId": "550e8400-e29b-41d4-a716-446655440000"
    }

Resultado:

    Pedido: ABERTO
    Mesa: OCUPADA

---

## 5. Garçom adiciona itens ao pedido

Endpoint:

    POST /pedidos/{pedidoId}/itens

Body:

    {
      "produtoId": "26f5c47f-40ef-46fd-8c70-6ec85a8f43b7",
      "quantidade": 2
    }

Resultado:

    Item adicionado ao pedido
    Valor total recalculado

---

## 6. Cozinha inicia o preparo

Endpoint:

    PATCH /pedidos/{id}/preparar

Resultado:

    Pedido: EM_PREPARO

---

## 7. Cozinha marca pedido como servido

Endpoint:

    PATCH /pedidos/{id}/servir

Resultado:

    Pedido: SERVIDO
    Mesa: AGUARDANDO_PAGAMENTO

---

## 8. Recepção confirma o pagamento

Endpoint:

    PATCH /pedidos/{id}/pagar

Resultado:

    Pedido: PAGO
    Mesa: FINALIZADA

---

## 9. Recepção libera a mesa

Endpoint:

    PATCH /mesas/{id}/liberar

Resultado:

    Mesa: DISPONIVEL

---

# Resumo dos Endpoints

## Mesas

    POST   /mesas
    GET    /mesas
    GET    /mesas/{id}
    PUT    /mesas/{id}
    DELETE /mesas/{id}
    PATCH  /mesas/{id}/liberar

---

## Categorias

    POST   /categorias
    GET    /categorias
    GET    /categorias/{id}
    PUT    /categorias/{id}
    DELETE /categorias/{id}

---

## Produtos

    POST   /produtos
    GET    /produtos
    GET    /produtos?categoriaId=4cd1d7a1-74a8-41e2-9876-b845cb9dbd91
    GET    /produtos/{id}
    PUT    /produtos/{id}
    DELETE /produtos/{id}

---

## Pedidos

    POST   /pedidos
    GET    /pedidos
    GET    /pedidos?status=ABERTO
    GET    /pedidos/{id}
    DELETE /pedidos/{id}
    PATCH  /pedidos/{id}/cancelar
    PATCH  /pedidos/{id}/preparar
    PATCH  /pedidos/{id}/servir
    PATCH  /pedidos/{id}/pagar

---

## Itens do Pedido

    POST   /pedidos/{pedidoId}/itens
    GET    /pedidos/{pedidoId}/itens
    PUT    /pedidos/{pedidoId}/itens/{itemId}
    DELETE /pedidos/{pedidoId}/itens/{itemId}

---

# Prioridade de Desenvolvimento

## Sprint 1 - Configuração Inicial

- [ ] Criar projeto Node.js
- [ ] Instalar Express
- [ ] Instalar dependências principais
- [ ] Configurar Docker Compose com PostgreSQL
- [ ] Configurar arquivo `.env`
- [ ] Configurar conexão com banco PostgreSQL
- [ ] Configurar UUID/GUID como padrão para todos os IDs
- [ ] Criar estrutura de pastas
- [ ] Criar servidor inicial
- [ ] Criar rota de health check

Endpoint recomendado:

    GET /health

Resposta:

    {
      "status": "API Restaurante funcionando"
    }

---

## Sprint 2 - Autenticação e Permissões

- [ ] Criar middleware de autenticação JWT
- [ ] Criar middleware de permissão por perfil
- [ ] Validar token recebido da API Auth PHP
- [ ] Garantir que o ID do usuário autenticado seja tratado como UUID/GUID
- [ ] Bloquear endpoints sem token
- [ ] Bloquear endpoints sem permissão
- [ ] Testar permissões dos perfis ADMIN, GARCOM, RECEPCAO e COZINHA

---

## Sprint 3 - CRUD de Mesas

- [ ] Criar model/tabela de mesas
- [ ] Definir `id` da mesa como UUID/GUID
- [ ] Criar endpoint para cadastrar mesa
- [ ] Criar endpoint para listar mesas
- [ ] Criar endpoint para buscar mesa por ID
- [ ] Criar endpoint para atualizar mesa
- [ ] Criar endpoint para excluir mesa
- [ ] Criar endpoint para liberar mesa
- [ ] Validar número único da mesa
- [ ] Validar status da mesa
- [ ] Bloquear exclusão de mesa ocupada

---

## Sprint 4 - CRUD de Categorias

- [ ] Criar model/tabela de categorias
- [ ] Definir `id` da categoria como UUID/GUID
- [ ] Criar endpoint para cadastrar categoria
- [ ] Criar endpoint para listar categorias
- [ ] Criar endpoint para buscar categoria por ID
- [ ] Criar endpoint para atualizar categoria
- [ ] Criar endpoint para excluir categoria
- [ ] Validar nome obrigatório
- [ ] Bloquear exclusão de categoria com produtos vinculados

---

## Sprint 5 - CRUD de Produtos

- [ ] Criar model/tabela de produtos
- [ ] Definir `id` do produto como UUID/GUID
- [ ] Definir `categoriaId` como UUID/GUID
- [ ] Criar relacionamento com categorias
- [ ] Criar endpoint para cadastrar produto
- [ ] Criar endpoint para listar produtos
- [ ] Criar filtro de produtos por categoria
- [ ] Criar endpoint para buscar produto por ID
- [ ] Criar endpoint para atualizar produto
- [ ] Criar endpoint para excluir produto
- [ ] Validar preço maior que zero
- [ ] Validar existência da categoria

---

## Sprint 6 - CRUD de Pedidos

- [ ] Criar model/tabela de pedidos
- [ ] Definir `id` do pedido como UUID/GUID
- [ ] Definir `mesaId` como UUID/GUID
- [ ] Criar relacionamento com mesas
- [ ] Criar endpoint para abrir pedido
- [ ] Alterar mesa para `OCUPADA` ao abrir pedido
- [ ] Criar endpoint para listar pedidos
- [ ] Criar filtro de pedidos por status
- [ ] Criar endpoint para buscar pedido por ID
- [ ] Criar endpoint para cancelar pedido
- [ ] Criar endpoint para excluir pedido
- [ ] Bloquear pedido para mesa ocupada
- [ ] Bloquear cancelamento fora do status `ABERTO`

---

## Sprint 7 - Itens do Pedido

- [ ] Criar model/tabela de itens do pedido
- [ ] Definir `id` do item como UUID/GUID
- [ ] Definir `pedidoId` como UUID/GUID
- [ ] Definir `produtoId` como UUID/GUID
- [ ] Criar relacionamento com pedido
- [ ] Criar relacionamento com produto
- [ ] Criar endpoint para adicionar item ao pedido
- [ ] Criar endpoint para listar itens do pedido
- [ ] Criar endpoint para atualizar quantidade do item
- [ ] Criar endpoint para remover item do pedido
- [ ] Copiar preço atual do produto para `valorUnitario`
- [ ] Recalcular `valorTotal` do pedido
- [ ] Bloquear alteração de itens se pedido não estiver `ABERTO`

---

## Sprint 8 - Fluxo de Status

- [ ] Criar endpoint para enviar pedido para preparo
- [ ] Criar endpoint para marcar pedido como servido
- [ ] Criar endpoint para marcar pedido como pago
- [ ] Criar validação de transições de status
- [ ] Atualizar mesa para `AGUARDANDO_PAGAMENTO` quando pedido for servido
- [ ] Atualizar mesa para `FINALIZADA` quando pedido for pago
- [ ] Criar endpoint para liberar mesa
- [ ] Bloquear transições inválidas

---

## Sprint 9 - Testes e Documentação

- [ ] Testar todos os endpoints no Postman ou Insomnia
- [ ] Criar collection da API
- [ ] Documentar endpoints
- [ ] Documentar bodies de requisição
- [ ] Documentar exemplos de resposta
- [ ] Documentar permissões por perfil
- [ ] Documentar fluxo de status dos pedidos
- [ ] Documentar fluxo de status das mesas
- [ ] Documentar uso do Docker Compose
- [ ] Documentar que todos os IDs devem ser UUID/GUID
- [ ] Criar README completo da API

---

# Validações Obrigatórias

## Mesa

- [ ] ID em UUID/GUID
- [ ] Número obrigatório
- [ ] Número único
- [ ] Status válido
- [ ] Não permitir exclusão de mesa ocupada
- [ ] Não permitir pedido em mesa indisponível

---

## Categoria

- [ ] ID em UUID/GUID
- [ ] Nome obrigatório
- [ ] Nome único, se possível
- [ ] Não permitir exclusão com produtos vinculados

---

## Produto

- [ ] ID em UUID/GUID
- [ ] CategoriaId em UUID/GUID
- [ ] Nome obrigatório
- [ ] Descrição obrigatória
- [ ] Preço obrigatório
- [ ] Preço maior que zero
- [ ] Categoria obrigatória
- [ ] Categoria existente

---

## Pedido

- [ ] ID em UUID/GUID
- [ ] MesaId em UUID/GUID
- [ ] Mesa obrigatória
- [ ] Mesa existente
- [ ] Mesa disponível
- [ ] Status inicial automático como `ABERTO`
- [ ] Valor total inicial como `0`
- [ ] Data de criação automática
- [ ] Transições de status válidas

---

## ItemPedido

- [ ] ID em UUID/GUID
- [ ] PedidoId em UUID/GUID
- [ ] ProdutoId em UUID/GUID
- [ ] Pedido obrigatório
- [ ] Produto obrigatório
- [ ] Quantidade obrigatória
- [ ] Quantidade maior que zero
- [ ] Valor unitário copiado automaticamente do produto
- [ ] Pedido precisa estar aberto
- [ ] Recalcular valor total após alteração

---

# Critérios de Pronto

A API Node.js será considerada pronta quando:

- [ ] O projeto estiver rodando localmente.
- [ ] O PostgreSQL estiver configurado via Docker Compose.
- [ ] A API estiver conectando corretamente ao banco PostgreSQL.
- [ ] Todos os IDs estiverem usando UUID/GUID.
- [ ] Todos os relacionamentos estiverem usando UUID/GUID.
- [ ] A autenticação via token da API Auth PHP estiver funcionando.
- [ ] As permissões por perfil estiverem funcionando.
- [ ] O CRUD de mesas estiver completo.
- [ ] O CRUD de categorias estiver completo.
- [ ] O CRUD de produtos estiver completo.
- [ ] O CRUD de pedidos estiver completo.
- [ ] O CRUD de itens do pedido estiver completo.
- [ ] O valor total do pedido estiver sendo calculado automaticamente.
- [ ] O fluxo `ABERTO -> EM_PREPARO -> SERVIDO -> PAGO` estiver funcionando.
- [ ] O fluxo `ABERTO -> CANCELADO` estiver funcionando.
- [ ] As mesas estiverem mudando de status conforme o pedido.
- [ ] As transições inválidas estiverem bloqueadas.
- [ ] A collection do Postman ou Insomnia estiver criada.
- [ ] O README estiver documentado.
- [ ] O projeto puder ser iniciado com Docker Compose e comandos simples.

---

# Divisão Recomendada de Responsabilidades

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
- Integração com token da API Auth PHP
- Conexão com PostgreSQL via Docker Compose
- Uso de UUID/GUID em todos os IDs e relacionamentos

---

## API Auth PHP

Responsável por:

- Cadastro de usuários
- Login
- Senhas
- Perfis
- Geração de token JWT
- Controle de usuários
- Envio do ID do usuário autenticado como UUID/GUID no token

---

## Frontend

Responsável por:

- Tela de login consumindo a API Auth PHP
- Tela de mesas
- Tela de cardápio
- Tela de criação de pedidos
- Tela da cozinha
- Tela da recepção
- Tela administrativa
- Consumo dos endpoints da API Node.js
- Envio dos IDs em formato UUID/GUID nas requisições

---

## QA

Responsável por testar:

- Login
- Permissões por perfil
- CRUD de mesas
- CRUD de produtos
- CRUD de categorias
- Abertura de pedidos
- Adição de itens
- Cálculo do valor total
- Fluxo de status dos pedidos
- Fluxo de status das mesas
- Bloqueios de ações inválidas
- Validação de UUID/GUID nos endpoints

---

# Fluxo Final Resumido

    Admin cadastra mesas
      ↓
    Admin cadastra categorias
      ↓
    Admin cadastra produtos
      ↓
    Garçom abre pedido para mesa disponível
      ↓
    Mesa muda para OCUPADA
      ↓
    Garçom adiciona itens ao pedido
      ↓
    Pedido calcula valor total
      ↓
    Cozinha altera pedido para EM_PREPARO
      ↓
    Cozinha altera pedido para SERVIDO
      ↓
    Mesa muda para AGUARDANDO_PAGAMENTO
      ↓
    Recepção confirma pagamento
      ↓
    Pedido muda para PAGO
      ↓
    Mesa muda para FINALIZADA
      ↓
    Recepção libera mesa
      ↓
    Mesa volta para DISPONIVEL
