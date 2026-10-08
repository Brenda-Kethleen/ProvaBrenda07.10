# Constituição — Zona Azul Digital

## 0. Modelo de correção
Kimi 2.8 — Preview 2026, sem chicote. Apenas o agente, com janela de 256k tokens , executa a tarefa proposta pelos seus .md

## 1. Ordem de prioridade

Em caso de conflito vale: 
constituição
spec
plan
testes
tasks

## 2. Idioma e nomes

- Código em inglês; documentação em português.
- Exceção: campos e rotas copiados do enunciado ficam literalmente como estão.


## 3. Convenções da API

- Recursos no plural; snake_case.
- IDs inteiros sequenciais a partir de 1.
-ISO 8601 com fuso -03:00.

## 4. Status codes

| Situação | Status |

| Criar: 201 |
| Consultar e listar: 200 |
| Remover / cancelar: 204 |
| Qualquer erro de validação: 422 em todos os erros de validação|
| Inexistente ou cancelado: 409 |
| Conflito: 409 |

- Formato do erro: `{"detail": "<codigo>"}`
- Precedência quando há mais de um erro: 400 > 404 > 409

## 5. Stack e contêiner

- Python 3.12 + FastAPI; persistência em memória.
- Um único contêiner Docker com a API, na porta 8003 minha variante.
- Dependências permitidas (lista fechada): [fastapi, uvicorn, pytest, httpx].

## 6. Encapsulamento

- Componentes: Rotas, Modelos, Serviço e Store. Cada um só é usado pela sua interface pública.
- Dependência em um sentido só: Rotas > Serviço > Store.
- Regra de negócio fica só no Serviço. Só o Store toca nos dados.

## 7. Artefatos obrigatórios

Todo código gerado inclui Dockerfile, README.md, requirements.txt e testes.

## 8. Regras de teste

- Um "def test_" por cenário do tests.
- Testes independentes entre si; reset do Store antes de cada um.

### variante para conhecimento:
{
  "slug": "ProvaBrenda07.10",
  "EXAM_DIR": "exams/2026/track-01-especificacao-sdd",
  "TARIFA_HORA_CENTAVOS": 500,
  "FRACAO_MINUTOS": 15,
  "TETO_DIARIO_CENTAVOS": 6000,
  "PORTA_SERVICO": 8003,
  "TOLERANCIA_MINUTOS": 15
}






