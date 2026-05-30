# Sistema de Pedidos em Restaurantes

<p align="center">
  <img src="https://img.shields.io/badge/Comunidade-Juninhos-7B2CBF?style=for-the-badge&logo=discord&logoColor=white" alt="Juninhos Community" />
  <img src="https://img.shields.io/badge/Status-EM%20ANDAMENTO-yellow?style=for-the-badge" alt="Status" />
</p>

---

## 📝 Sobre o Projeto

O Sistema de Pedidos em Restaurantes é uma solução digital desenvolvida para automatizar o fluxo de atendimento em estabelecimentos gastronômicos, facilitando a comunicação entre recepção, garçons, cozinha e administração.

O projeto está sendo desenvolvido de forma colaborativa pelos membros da Comunidade Juninhos com foco na aplicação de boas práticas de arquitetura de software, desenvolvimento web, mobile, APIs REST, autenticação e integração entre sistemas.

---

## 🎯 Objetivo

Centralizar e otimizar o gerenciamento do restaurante através de uma plataforma composta por:

* Frontend Web para administração, recepção e cozinha;
* Aplicativo Android para garçons;
* APIs independentes para autenticação e gerenciamento operacional;
* Banco de dados PostgreSQL executado em containers Docker.

---

## 🛠️ Stack Tecnológica

### Frontend

* Angular

### Mobile

* Kotlin
* Jetpack Compose
* Retrofit

### Backend

* Node.js
* PHP

### Banco de Dados

* PostgreSQL

### Infraestrutura

* Docker
* Docker Compose

### Qualidade

* Cypress
* Postman
* Insomnia

---

## 🏗️ Arquitetura da Solução

```text
Frontend Angular
        │
        ▼
 ┌─────────────┐
 │ Auth API    │ (PHP)
 └─────────────┘
        │
        ▼
 ┌─────────────┐
 │ Restaurante │ (Node.js)
 │ API         │
 └─────────────┘
        │
        ▼
 PostgreSQL
```

```text
Aplicativo Kotlin
        │
        ├── Auth API (PHP)
        │
        └── Restaurante API (Node.js)
```

---

## 👤 Perfis de Usuário

### Admin

Responsável por:

* Visualizar dashboard gerencial;
* Consultar faturamento diário;
* Consultar pratos mais vendidos;
* Acompanhar indicadores operacionais.

### Recepção

Responsável por:

* Consultar mesas disponíveis;
* Consultar mesas ocupadas;
* Consultar mesas aguardando pagamento;
* Acompanhar status geral das mesas.

### Cozinha

Responsável por:

* Visualizar pedidos recebidos;
* Atualizar andamento dos pedidos;
* Gerenciar cardápio;
* Cadastrar, editar e remover produtos.

### Garçom

Responsável por:

* Abrir pedidos;
* Editar pedidos;
* Cancelar pedidos;
* Alterar status das mesas;
* Finalizar atendimento.

---

## 📌 Funcionalidades Principais

* [ ] 🔐 Autenticação de usuários com controle de acesso por perfil (Admin, Recepção, Cozinha e Garçom).

* [ ] 🍽️ Gestão operacional do restaurante através da API Node.js.

* [ ] 🖥️ Frontend Angular para administração, recepção e cozinha.

* [ ] 📱 Aplicativo Android em Kotlin para operação dos garçons.

---

## 📊 Dashboard Administrativo

O painel administrativo deverá apresentar:

* Total faturado no dia;
* Quantidade de pedidos realizados;
* Quantidade de mesas ocupadas;
* Pratos mais vendidos.

---

## 🧱 Entidades Principais

### Auth API (PHP)

#### Usuário

```text
Id
Nome
Email
Senha
Perfil
DataCriacao
```

#### Perfil

```text
ADMIN
RECEPCAO
COZINHA
GARCOM
```

---

### Restaurante API (Node.js)

#### Mesa

```text
Id
Numero
Status
```

#### Categoria

```text
Id
Nome
```

#### Produto

```text
Id
Nome
Descricao
Preco
CategoriaId
```

#### Pedido

```text
Id
MesaId
Status
ValorTotal
DataCriacao
```

#### ItemPedido

```text
Id
PedidoId
ProdutoId
Quantidade
ValorUnitario
```

---

## 🚦 Status das Mesas

```text
DISPONIVEL
OCUPADA
AGUARDANDO_PAGAMENTO
FINALIZADA
```

---

## 🚦 Status dos Pedidos

```text
ABERTO
EM_PREPARO
SERVIDO
PAGO
CANCELADO
```

### Fluxo Principal

```text
ABERTO
    ↓
EM_PREPARO
    ↓
SERVIDO
    ↓
PAGO
```

### Fluxo de Cancelamento

```text
ABERTO
    ↓
CANCELADO
```

---

## ⚙️ Como Executar o Projeto Localmente

### Pré-requisitos

* Git
* Docker
* Docker Compose
* Node.js
* PHP
* Android Studio
* Angular CLI

### Clone do Projeto

```bash
git clone https://github.com/juninhos-comunidade/Sistema-de-pedidos-em-restaurante.git
```

```bash
cd Sistema-de-pedidos-em-restaurante
```

### Inicializar Infraestrutura

```bash
docker compose up -d
```

### Executar Frontend

```bash
cd frontend
npm install
ng serve
```

### Executar API Node

```bash
cd api-restaurante
npm install
npm run dev
```

### Executar API PHP

```bash
cd api-auth
composer install
php artisan serve
```

### Executar Aplicativo Android

Abrir o projeto Android Studio e executar em dispositivo físico ou emulador.

---

## 🌿 Git Flow

### Branches

```text
main
feature/*
fix/*
docs/*
```

Exemplos:

```bash
git checkout -b feature/pedidos
```

```bash
git checkout -b feature/dashboard
```

```bash
git checkout -b fix/correcao-login
```

---

## 📝 Convenção de Commits

```text
feat: nova funcionalidade
fix: correção de bug
docs: documentação
style: ajustes visuais
refactor: refatoração
test: testes
chore: tarefas de manutenção
```

Exemplos:

```bash
feat: adiciona CRUD de pedidos
```

```bash
fix: corrige autenticação JWT
```

```bash
docs: atualiza README
```

---

## 🚀 Fluxo de Demonstração

1. Recepção visualiza mesas disponíveis.
2. Garçom realiza login no aplicativo.
3. Garçom seleciona uma mesa.
4. Garçom cria um pedido.
5. Pedido é enviado para a cozinha.
6. Cozinha altera status para EM_PREPARO.
7. Cozinha altera status para SERVIDO.
8. Garçom finaliza o atendimento.
9. Pedido é marcado como PAGO.
10. Dashboard administrativo é atualizado automaticamente.

---

## 👥 Nosso Squad

|                         Avatar                        | Membro       | Função / Especialidade                    | GitHub                                                     |
| :---------------------------------------------------: | :----------- | :---------------------------------------- | :--------------------------------------------------------- |
| <img src="https://github.com/github.png" width="40"/> | Miguel Rocha | Tech Lead e Desenvolvedor Frontend/Mobile | [@rocha-miguel](https://github.com/rocha-miguel)           |
| <img src="https://github.com/github.png" width="40"/> | Ryan         | Desenvolvedor Backend Node.js             | [@MrRyan04](https://github.com/MrRyan04)                   |
| <img src="https://github.com/github.png" width="40"/> | Edimilson    | Desenvolvedor Backend                     | [@Edimilson-Miranda](https://github.com/Edimilson-Miranda) |
| <img src="https://github.com/github.png" width="40"/> | Thiago       | Desenvolvedor Backend PHP                 | [@ThiagoRosaPaiva](https://github.com/ThiagoRosaPaiva)     |
| <img src="https://github.com/github.png" width="40"/> | Alícia       | Analista de QA                            | [@Ali-Maia](https://github.com/Ali-Maia)                   |

---

## ⚖️ Licença

Projeto desenvolvido exclusivamente para fins educacionais e colaborativos dentro da Comunidade Juninhos.

---

## 🤝 Comunidade Juninhos

Projeto desenvolvido pelos membros da Comunidade Juninhos com foco em aprendizado, colaboração e evolução profissional através de experiências práticas de desenvolvimento de software.
