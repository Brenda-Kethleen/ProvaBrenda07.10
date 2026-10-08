# Spec — Zona Azul Digital

API REST de bilhetes de estacionamento rotativo. URL base: `http://localhost:8003`

## 1. Parâmetros

| Parâmetro | Valor |
| --- | --- |
| `TARIFA_HORA_CENTAVOS` | 500 |
| `FRACAO_MINUTOS` | 15 (fração = 125 centavos) |
| `TETO_DIARIO_CENTAVOS` | 6000 |
| `TOLERANCIA_MINUTOS` | 15 |
| `PORTA_SERVICO` | 8003 |

## 2. Bilhete

- Aberto ou cancelado: `id`, `placa`, `entrada`, `status`.
- Encerrado em listagens: os anteriores + `saida`, `minutos`, `valor_centavos`.
- Resposta do UC2: `id`, `placa`, `entrada`, `saida`, `minutos`, `valor_centavos` (sem `status`).
- `status`: `"aberto"`, `"encerrado"` ou `"cancelado"`. Campo não aplicável é omitido, nunca `null`.
- `id` inteiro sequencial a partir de 1. Datas em ISO-8601 com offset `-03:00`.
- "Mais recentes primeiro" = `entrada` decrescente; empate por `id` decrescente.

## 3. Regra de valor

1. `minutos` = segundos entre `entrada` e `saida` ÷ 60, truncado; mínimo 0.
2. `minutos` ≤ 15 → `valor_centavos` 0.
3. Senão: frações = `minutos` ÷ 15 arredondado para cima (tolerância não é descontada); valor = frações × 125.

> [!WARNING]
> Teto: `valor_centavos` nunca passa de 6000 (atingido em 720 min; 721 continua 6000).

## 4. Casos de uso

### UC1 — Abrir bilhete

`POST /bilhetes`, corpo `{"placa": "ABC1D23"}`, com `entrada` opcional → **201**
com o bilhete aberto. Ordem de verificação: `placa_invalida`, depois
`entrada_invalida`, depois `bilhete_em_aberto`.

- CA1.1: `{"placa": "ABC1D23"}` → 201 com `id` inteiro, `placa` igual à
  enviada, `entrada` terminando em `-03:00` e `status` igual a `"aberto"`.
- CA1.2: o primeiro bilhete criado tem `id` 1 e o segundo tem `id` 2.
- CA1.3: com `"entrada": "2026-10-05T10:00:00-03:00"` → 201 e `entrada` idêntica na resposta.
- CA1.4: com `"entrada": "ontem"` → 422 `{"erro": "entrada_invalida"}`.
- CA1.5: placa `"abc1d23"`, `"ABC1D2"` ou ausente → 422 `{"erro": "placa_invalida"}`.

### UC2 — Encerrar bilhete

`POST /bilhetes/{id}/encerramento` → **200** com exatamente `id`, `placa`,
`entrada`, `saida`, `minutos`, `valor_centavos`. `saida` é o instante atual.

- CA2.1: entrada 30 minutos antes → `minutos` 30, `valor_centavos` 250.
- CA2.2: entrada 31 minutos antes → `minutos` 31, `valor_centavos` 375.
- CA2.3: entrada 721 minutos antes → `valor_centavos` 6000.
- CA2.4: `minutos` e `valor_centavos` são inteiros JSON, sem ponto decimal.
- CA2.5: `id` 999 inexistente → 404 `{"erro": "bilhete_nao_encontrado"}`.
- CA2.6: bilhete já encerrado ou cancelado → 409 `{"erro": "bilhete_ja_encerrado"}`.

### UC3 — Listar ativos

`GET /bilhetes/ativos` → **200** com array dos bilhetes de status `"aberto"`,
mais recentes primeiro.

- CA3.1: sem bilhetes abertos → 200 `[]`.
- CA3.2: abertos A e depois B → array `[B, A]`.
- CA3.3: bilhete encerrado ou cancelado não aparece.

### UC4 — Relatório diário

`GET /relatorios/diario?data=AAAA-MM-DD` → **200** com `data`,
`total_bilhetes`, `faturamento_centavos`, `tempo_medio_minutos`.

"Bilhetes do dia" = bilhetes cuja `entrada`, em `-03:00`, cai na data pedida.

- `total_bilhetes`: quantidade de bilhetes do dia, em qualquer status.
- `faturamento_centavos`: soma de `valor_centavos` dos bilhetes do dia encerrados.
- `tempo_medio_minutos`: média de `minutos` dos bilhetes do dia encerrados,
  arredondando 0,5 para cima; sem encerrados → 0.

- CA4.1: dia com encerrados de 30 min (250) e 61 min (625) e um aberto →
  `total_bilhetes` 3, `faturamento_centavos` 875, `tempo_medio_minutos` 46 (45,5 sobe).
- CA4.2: dia sem bilhetes → 200 com os três valores iguais a 0 e `data` igual à pedida.
- CA4.3: `data=05/10/2026` ou sem `data` → 422 `{"erro": "data_invalida"}`.

### UC5 — Cancelar bilhete

`POST /bilhetes/{id}/cancelamento` → **200** com o bilhete em `status`
`"cancelado"`. Sem cobrança.

- CA5.1: bilhete aberto → 200 com `id`, `placa`, `entrada`, `status`
  `"cancelado"`; sem `saida` e sem `valor_centavos`.
- CA5.2: bilhete já cancelado ou já encerrado → 409 `{"erro": "bilhete_nao_aberto"}`.
- CA5.3: `id` inexistente → 404 `{"erro": "bilhete_nao_encontrado"}`.

### UC6 — Histórico por placa

`GET /bilhetes?placa=ABC1D23` → **200** com array de todos os bilhetes da placa,
em qualquer status, mais recentes primeiro.

- CA6.1: placa com um bilhete encerrado, um cancelado e um aberto → array com 3 itens.
- CA6.2: placa válida que nunca estacionou → 200 `[]`.
- CA6.3: `placa` ausente ou inválida → 422 `{"erro": "placa_invalida"}`.

### UC7 — Tolerância gratuita

Duração ≤ 15 minutos → `valor_centavos` 0. Acima disso, cobrança integral desde
o primeiro minuto.

- CA7.1: 15 minutos → `valor_centavos` 0.
- CA7.2: 16 minutos → `valor_centavos` 250 (2 frações, nunca 125).
- CA7.3: 0 minutos → `valor_centavos` 0.

### UC8 — Uma vaga por placa

`POST /bilhetes` para placa com bilhete aberto → **409** `{"erro": "bilhete_em_aberto"}`.

- CA8.1: segundo `POST` com a mesma placa → 409.
- CA8.2: após encerrar o bilhete, novo `POST` com a mesma placa → 201.
- CA8.3: após cancelar o bilhete, novo `POST` com a mesma placa → 201.
- CA8.4: `POST` com outra placa enquanto a primeira está aberta → 201.

## 5. Erros

Situação	Status	Corpo
Placa de mercado ou inválida	422	{"erro": "placa_invalida"}
entradafora de ISO-8601	422	{"erro": "entrada_invalida"}
datafora deAAAA-MM-DD	422	{"erro": "data_invalida"}
Bilhete inexistente	404	{"erro": "bilhete_nao_encontrado"}
Fechar bilhete já encerrado	409	{"erro": "bilhete_ja_encerrado"}
Cancelar bilhete não aberto	409	{"erro": "bilhete_nao_aberto"}
Abrir bilhete com placa já ocupada	409	{"erro": "bilhete_em_aberto"}

Precedência: 422 > 404 > 409.