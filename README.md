# Desafio Técnico — Frontend Angular (Júnior)

## Sobre o desafio

Você vai desenvolver o frontend de um painel de acompanhamento de entregas.

Pedidos chegam ao longo do dia, mudam de estado conforme são preparados e entregues, e cada um possui um horário prometido ao cliente.

O objetivo deste desafio é avaliar seus conhecimentos em Angular, TypeScript, consumo de APIs, organização de código e sua capacidade de manter a interface consistente enquanto os dados mudam em tempo real.

Não avaliamos design. O foco está na qualidade técnica da solução.

**Prazo:** 5 dias corridos  
**Esforço esperado:** 5 a 7 horas

Se algo não couber no tempo disponível, entregue o que foi feito e descreva no README o que ficaria para uma próxima etapa.

Um projeto menor e bem resolvido vale mais do que um projeto maior incompleto.

---

# Stack

Tecnologias obrigatórias:

- Angular 22
- TypeScript (`strict: true`)
- pnpm
- SCSS

Requisitos técnicos:

- Standalone Components
- Angular Signals
- Angular Router

Não utilize:

- Angular Material
- PrimeNG
- NgRx
- NGXS
- Akita
- Bibliotecas de gerenciamento global de estado

CSS próprio é suficiente.

---

# Requisitos de Ambiente

- Node.js 24+
- pnpm

---

# API

Uma API já hospedada será disponibilizada juntamente com este desafio.

Não é necessário executar backend localmente.

**Base URL**

```txt
https://orders-api.planumlabs.com
```

**Documentação da API**

```txt
https://orders-api.planumlabs.com/swagger-ui/index.html
```

---

## Tratamento de Erros

Erros seguem o formato:

```json
{
  "timestamp": "2026-09-11T14:22:31.000-03:00",
  "status": 409,
  "error": "Conflict",
  "message": "O pedido já está em EM_ROTA",
  "path": "/pedidos/812/transicoes"
}
```

---

## GET /pedidos

Lista os pedidos com paginação, filtro e ordenação.

### Parâmetros

| Parâmetro | Descrição |
|------------|------------|
| page | Página atual |
| size | Quantidade por página |
| status | Filtrar por status |
| busca | Buscar por cliente |
| ordenarPor | criadoEm ou prometidoPara |
| ordem | asc ou desc |

### Exemplo de Resposta

```json
{
  "servidorEm": "2026-09-11T14:22:31.000-03:00",
  "conteudo": [
    {
      "id": 812,
      "codigo": "PED-0812",
      "clienteNome": "Maria Silva",
      "enderecoResumo": "Rua das Acácias, 210 — Pinheiros",
      "status": "EM_PREPARO",
      "valorTotal": 87.40,
      "criadoEm": "2026-09-11T14:02:00.000-03:00",
      "prometidoPara": "2026-09-11T14:47:00.000-03:00",
      "versao": 3
    }
  ],
  "pagina": 1,
  "tamanho": 20,
  "total": 143,
  "totalPaginas": 8
}
```

---

## GET /operacao/stream

Endpoint Server-Sent Events (SSE).

Eventos disponíveis:

### pedido.criado

```text
event: pedido.criado
```

### pedido.transicionado

```text
event: pedido.transicionado
```

### heartbeat

```text
event: heartbeat
```

Exemplo:

```text
id: 4471
event: pedido.criado
data: {...}

id: 4472
event: pedido.transicionado
data: {...}

id: 4473
event: heartbeat
data: {...}
```

O heartbeat é enviado a cada 10 segundos.

A conexão será encerrada periodicamente de forma proposital para simular falhas de rede.

O navegador já gerencia automaticamente a reconexão utilizando `EventSource`.

Não é necessário implementar um mecanismo próprio de reconexão.

---

## POST /pedidos/{id}/transicoes

Altera o estado de um pedido.

### Exemplo

```json
{
  "para": "EM_ROTA",
  "motivo": null
}
```

### Respostas

| Código | Descrição |
|----------|------------|
| 200 | Transição aplicada com sucesso |
| 409 | Pedido já mudou de estado |
| 422 | Transição inválida ou motivo ausente |

O backend possui um atraso artificial entre 300ms e 1s para simular ambiente real.

---

# Fluxo de Estados

```text
RECEBIDO ──> EM_PREPARO ──> PRONTO ──> EM_ROTA ──> ENTREGUE
    │             │
    └─────────────┴──────────> CANCELADO
```

Regras:

- CANCELADO exige motivo com no mínimo 10 caracteres
- ENTREGUE é estado final
- CANCELADO é estado final
- Qualquer outra transição é inválida

A interface deve exibir apenas as ações válidas para o estado atual.

---

# Telas

## 1. /operacao

Tela de acompanhamento em tempo real.

### Requisitos

- Exibir todos os pedidos ativos
- Excluir ENTREGUE e CANCELADO
- Atualizar automaticamente via SSE
- Exibir:
  - Código
  - Cliente
  - Status
  - Tempo restante até prometidoPara

### Contagem regressiva

A contagem deve atualizar a cada segundo.

Exibir destaque visual:

- Quando faltarem menos de 5 minutos
- Quando o prazo já tiver expirado

### Ações

Permitir transições conforme o estado atual.

### Conectividade

Exibir indicador visual:

- Conectado
- Reconectando

---

## 2. /pedidos

Histórico de pedidos.

### Requisitos

Tabela paginada contendo:

- Código
- Cliente
- Status
- Data de criação
- Prazo prometido

### Funcionalidades

- Paginação server-side
- Filtro por status
- Busca por cliente
- Ordenação

Importante:

A API já executa paginação, filtro e ordenação.

Não carregue todos os registros para processar no navegador.

### URL

A página atual e os filtros devem permanecer sincronizados com a URL.

Ao atualizar a página, o estado deve ser restaurado.

### Busca

Não realizar uma requisição a cada tecla digitada.

---

# Pontos de Atenção

Estes são os critérios mais importantes da avaliação.

---

## 1. Hora do Servidor

Não utilize apenas `Date.now()` para calcular o tempo restante.

O relógio do usuário pode estar incorreto.

Utilize o campo:

```json
{
  "servidorEm": "..."
}
```

Calcule a diferença entre o horário do servidor e o horário local uma única vez e utilize essa diferença em todos os cálculos.

Os horários devem ser exibidos sempre considerando o fuso horário de São Paulo.

---

## 2. Eventos Duplicados

Quando a conexão SSE for restabelecida, eventos podem ser reenviados.

Utilize o campo:

```json
{
  "versao": 4
}
```

Eventos com versão menor ou igual à já aplicada devem ser ignorados.

---

## 3. O Servidor é a Fonte da Verdade

Ao realizar uma transição:

- A interface pode ser atualizada imediatamente
- Caso a API retorne 409, o estado deve ser corrigido para refletir o estado informado pelo servidor

Também garanta que um clique duplo não envie múltiplas requisições.

---

## 4. Centralização das Regras

Evite espalhar verificações de status pelos templates.

Prefira uma estrutura centralizada responsável por responder:

> Quais transições são válidas para este estado?

---

## 5. Navegação por Teclado

Todos os botões devem:

- Ser acessíveis por teclado
- Possuir foco visível

Não é necessário implementar requisitos avançados de acessibilidade.

---

# Decisão Arquitetural

Considere o cenário:

O usuário abre o formulário de cancelamento de um pedido.

Enquanto digita o motivo, chega um evento SSE informando que o pedido mudou de estado.

Decida como sua aplicação irá reagir.

Implemente a solução escolhida e explique sua decisão no README.

Não existe uma resposta única correta.

Queremos entender seu raciocínio.

---

# Testes

Os seguintes testes unitários são obrigatórios:

1. Cálculo do tempo restante
2. Regras de transição entre estados

Testes adicionais são opcionais.

---

# Entrega

Publicar a solução em um repositório GitHub.

O projeto deve conter:

- Código-fonte
- README.md
- Instruções de instalação
- Instruções de execução
- Instruções para execução dos testes

---

# README do Projeto

No README entregue, responda brevemente:

### 1.

Como foi implementado o cálculo baseado em `servidorEm`.

### 2.

Como foi evitada a aplicação duplicada de eventos.

### 3.

Como filtros e paginação foram sincronizados com a URL.

### 4.

Qual decisão foi tomada para o cenário de conflito durante o cancelamento.

### 5.

O que ficou de fora e o que seria implementado com mais tempo.

---

# Diferenciais

Nenhum dos itens abaixo é obrigatório.

Priorize a entrega das duas telas principais.

- Utilização de `httpResource`
- Interceptor para tratamento global de erros
- Tela de detalhe do pedido
- Linha do tempo de eventos
- Tema claro e escuro
- Docker
- Deploy online

---

# O que Esperamos Ver

- Componentização adequada
- Uso correto de Signals
- Boas práticas Angular
- Consumo de API tipado
- Tratamento de erros
- Organização de código
- Responsividade básica
- Código legível e de fácil manutenção

---

# Critérios de Avaliação

| Peso | Critério |
|--------|-----------|
| 20% | Angular e uso de Signals |
| 20% | Organização e componentização |
| 20% | Tratamento correto da hora do servidor |
| 15% | Atualização em tempo real via SSE |
| 15% | Tratamento de erros (409 e 422) |
| 10% | Paginação, filtros e sincronização com URL |

---

# O que Não Avaliamos

- Design visual
- Paleta de cores
- Pixel perfect
- Framework CSS utilizado
- Quantidade de testes
- Cobertura de casos não descritos no enunciado

---

# Dúvidas

Se algum ponto não estiver claro, entre em contato.

Perguntar não desconta nota.

Ficar bloqueado por uma dúvida simples pode prejudicar sua entrega desnecessariamente.
