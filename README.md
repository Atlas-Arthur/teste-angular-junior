# Angular Junior Technical Challenge

## Sobre o desafio

O objetivo deste desafio é avaliar conhecimentos em Angular, TypeScript, consumo de APIs REST, componentização, organização de código e boas práticas de desenvolvimento frontend.

O candidato deverá desenvolver uma aplicação web utilizando Angular.

---

## Prazo

5 dias corridos a partir do recebimento deste desafio.

---

## Tecnologias obrigatórias

- Angular 22
- TypeScript
- pnpm
- SCSS

---

## Objetivo

Desenvolver uma aplicação para consulta de usuários consumindo uma API REST disponibilizada pela empresa.

---

## Funcionalidades

### Listagem de Usuários

Exibir uma tabela contendo:

- Nome
- E-mail
- Telefone
- Empresa

### Paginação

A listagem deve consumir paginação disponibilizada pela API.

Exemplo:

GET /users?page=1&pageSize=10

### Filtro

Permitir busca por nome.

Exemplo:

GET /users?search=john

### Ordenação

Permitir ordenação por nome.

Exemplo:

GET /users?sort=name&order=asc

### Detalhes do Usuário

Ao selecionar um usuário, exibir:

- Nome
- E-mail
- Telefone
- Endereço
- Empresa

---

## Requisitos Técnicos

A aplicação deverá:

- Utilizar Angular Router
- Utilizar Services para comunicação com a API
- Possuir tipagem TypeScript adequada
- Tratar cenários de erro
- Exibir estado de carregamento
- Possuir layout responsivo

---

## Diferenciais

Os itens abaixo não são obrigatórios, mas serão considerados diferenciais:

- Angular Signals
- Testes unitários
- Interceptors
- Docker
- Deploy online
- Dark mode

---

## Critérios de Avaliação

Serão avaliados os seguintes aspectos:

- Organização do código
- Componentização
- Boas práticas Angular
- TypeScript
- Consumo de API
- Tratamento de erros
- Responsividade
- Legibilidade do código

---

## Entrega

Publicar a solução em um repositório GitHub e compartilhar o link para avaliação.

---

## README do Projeto

O repositório entregue deve conter instruções para:

- Instalação das dependências
- Execução do projeto
- Decisões técnicas relevantes adotadas durante o desenvolvimento

---

## API

As informações da API serão disponibilizadas juntamente com este desafio.


---

## O que não será avaliado

- Frameworks CSS específicos
- Bibliotecas de componentes
- Identidade visual
- Pixel perfect

O foco da avaliação está na qualidade técnica da solução.
