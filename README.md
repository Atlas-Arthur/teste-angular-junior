# Desafio Técnico — Frontend Angular (Júnior)

## Sobre o desafio

Você vai desenvolver o frontend de um **painel de acompanhamento de entregas**. Pedidos chegam ao longo do dia, mudam de estado conforme são preparados e entregues, e cada um tem um horário prometido ao cliente.

O objetivo é avaliar como você lida com uma tela que precisa se manter correta enquanto os dados mudam no servidor. Não avaliamos design, então não gaste tempo com isso.

**Prazo:** 5 dias corridos.
**Esforço esperado:** 5 a 7 horas.

Se algo não couber no tempo, entregue o que fez e escreva no README o que ficaria para depois. Um projeto menor e bem resolvido vale mais que um maior pela metade.

---

## Stack

- Angular 22
- TypeScript com `strict: true`
- pnpm
- SCSS

A aplicação deve usar componentes standalone e **signals** para o estado. Aplicações novas do Angular 22 já nascem assim, então basta seguir o padrão do `ng new`.

Não use biblioteca de componentes prontos (Material, PrimeNG). CSS próprio é suficiente. Não usamos NgRx nem nada parecido aqui, então não precisa.

---

## A API

Você recebe um servidor de mock junto com o desafio, com instruções de execução.

Erros vêm sempre neste formato:

```json
{
  "timestamp": "2026-09-11T14:22:31.000-03:00",
  "status": 409,
  "error": "Conflict",
  "message": "O pedido já está em EM_ROTA",
  "path": "/pedidos/812/transicoes"
}
```

Toda resposta inclui o campo **`servidorEm`**, com a hora atual do servidor. Você vai usar esse campo — a seção "Pontos de atenção" explica por quê.

### `GET /pedidos`

Parâmetros: `page`, `size` (padrão 20), `status`, `busca`, `ordenarPor` (`prometidoPara` ou `criadoEm`), `ordem` (`asc` ou `desc`).

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

### `GET /operacao/stream` — Server-Sent Events

Emite as mudanças em tempo real. Cada evento tem um `id` numérico crescente.

```
id: 4471
event: pedido.criado
data: {"servidorEm":"...","pedido":{ ... }}

id: 4472
event: pedido.transicionado
data: {"servidorEm":"...","pedidoId":812,"para":"PRONTO","versao":4}

id: 4473
event: heartbeat
data: {"servidorEm":"2026-09-11T14:22:41.000-03:00"}
```

O `heartbeat` chega a cada 10 segundos. A conexão **cai de propósito** a cada poucos minutos, para você lidar com reconexão.

O `EventSource` nativo do navegador já reconecta sozinho e já reenvia o último `id` recebido. Você não precisa implementar isso na mão, mas precisa entender o que acontece.

### `POST /pedidos/{id}/transicoes`

```json
{ "para": "EM_ROTA", "motivo": null }
```

| Código | Situação |
|---|---|
| `200` | Transição aplicada. Retorna o pedido atualizado. |
| `409` | O pedido já mudou de estado. A `message` informa o estado atual. |
| `422` | Transição inválida, ou motivo ausente quando obrigatório. |

O mock demora entre 300ms e 1s de propósito.

---

## Estados do pedido

```
RECEBIDO ──> EM_PREPARO ──> PRONTO ──> EM_ROTA ──> ENTREGUE
    │             │
    └─────────────┴──────────> CANCELADO
```

- `→ CANCELADO` exige `motivo` com no mínimo 10 caracteres.
- `ENTREGUE` e `CANCELADO` são estados finais.
- Qualquer outra transição é inválida.

A tela só deve oferecer as ações permitidas pelo estado atual do pedido.

---

## Telas

### 1. `/operacao` — Acompanhamento ao vivo

- Lista dos pedidos ativos (todos menos `ENTREGUE` e `CANCELADO`), atualizada pelo SSE.
- Cada pedido mostra código, cliente, status e **quanto tempo falta até `prometidoPara`**, atualizando a cada segundo.
- Destaque visual quando faltam menos de 5 minutos e quando o prazo já passou.
- Botões de transição conforme o estado.
- Um indicador mostrando se a conexão está ativa ou reconectando.

### 2. `/pedidos` — Histórico

- Tabela paginada, com filtro por status, busca por nome do cliente e ordenação.
- **A API já faz paginação, filtro e ordenação.** Use os parâmetros dela, não carregue tudo e filtre no navegador.
- O filtro e a página atual devem aparecer na URL, de forma que recarregar a página mantenha o que estava sendo visto.
- A busca não deve disparar uma requisição a cada tecla digitada.

---

## Pontos de atenção

Estes são os pontos que mais pesam na avaliação. Nenhum deles é difícil, mas todos são fáceis de esquecer.

### 1. Use a hora do servidor, não a do navegador

O relógio do computador do usuário pode estar errado. Se a contagem regressiva usar `Date.now()` direto, ela mostra o número errado e ninguém percebe.

O servidor manda `servidorEm` em toda resposta e a cada `heartbeat`. Compare com a hora local uma vez, guarde a diferença, e use essa diferença em todos os cálculos de tempo.

Você pode testar mudando o relógio do seu sistema operacional. Se a contagem continuar certa, funcionou.

Exiba os horários sempre no fuso de São Paulo, independentemente do fuso configurado no navegador.

### 2. Um evento não pode ser aplicado duas vezes

Quando a conexão cai e volta, o servidor reenvia o que você perdeu. Nessa janela é possível receber de novo algo que você já aplicou.

O campo `versao` do pedido serve para isso: se chegar um evento com versão menor ou igual à que você já tem, ignore.

### 3. O servidor é quem manda

Quando o usuário clicar num botão de transição, você pode atualizar a tela na hora, sem esperar a resposta. Mas se a resposta vier `409`, significa que o pedido já estava em outro estado, e a tela precisa se corrigir para o estado que o servidor informou.

Também garanta que um clique duplo não envie a mesma ação duas vezes.

### 4. As regras de transição num lugar só

Espalhar `@if (pedido.status === 'PRONTO')` pelos templates funciona, mas vira problema quando surge um estado novo. Prefira uma estrutura que responda "quais ações são possíveis a partir deste estado", e use ela nos componentes.

### 5. Navegação por teclado

Todos os botões devem ser acionáveis por teclado, com foco visível. Não precisa ir além disso.

---

## Uma decisão para você tomar

O usuário abre o formulário de cancelamento de um pedido e, enquanto digita o motivo, chega pelo SSE a informação de que aquele pedido já mudou de estado.

Decida o que sua aplicação faz nesse caso, implemente, e explique a escolha no README. Não existe resposta certa única — queremos entender seu raciocínio.

---

## Testes

Dois testes unitários são suficientes:

1. O cálculo do tempo restante, incluindo o caso em que o prazo já passou.
2. A função que decide quais transições são possíveis a partir de um estado.

Se sobrar tempo e vontade, testes de componente são bem-vindos, mas não são esperados.

---

## Entrega

Repositório no GitHub com:

- Instruções de instalação e execução com pnpm
- Como rodar os testes
- `README.md` com suas decisões técnicas

No README, responda com algumas frases cada:

1. Como você usou o `servidorEm` para corrigir a contagem regressiva.
2. Como você garantiu que um evento não é aplicado duas vezes.
3. Como o filtro e a página ficam guardados na URL.
4. A decisão da seção anterior, e por quê.
5. O que ficou de fora e o que você faria com mais tempo.

Não precisa ser longo. Precisa ser claro sobre o que você pensou.

---

## Diferenciais

Nenhum é obrigatório. Entregue as duas telas primeiro.

- `httpResource` para carregar o histórico, com tratamento de carregando e erro
- Interceptor traduzindo o corpo de erro padronizado numa mensagem amigável
- Tela de detalhe do pedido com a linha do tempo de eventos
- Tema claro e escuro
- Docker ou deploy online

---

## O que avaliamos

| Peso | Critério |
|---|---|
| 25% | Tratamento correto do tempo: hora do servidor e fuso |
| 20% | Atualização em tempo real sem duplicar evento |
| 20% | Reação correta ao `409` e ao `422`, sem envio duplo |
| 15% | Regras de transição centralizadas e tipagem sem `any` |
| 10% | Uso da paginação e do filtro da API, e sincronia com a URL |
| 10% | Organização, componentização e legibilidade |

## O que não avaliamos

- Design, paleta, animação, pixel perfect
- Framework ou metodologia de CSS
- Quantidade de testes
- Cobertura de casos que não estão no enunciado

---

## Se travar

Se algum ponto do enunciado não estiver claro, escreva para nós. Perguntar não desconta nota. Entregar algo travado por causa de uma dúvida que a gente responderia em dois minutos, sim.
