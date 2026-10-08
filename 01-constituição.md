# Constituição — Zona Azul Digital

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

- Recursos no plural; JSON em camelCase.
- IDs inteiros sequenciais a partir de 1.
- Datas em ISO 8601 sem fuso.

## 4. Status codes

| Situação | Status |

| Criar: 201 |
| Consultar e listar: 200 |
| Remover / cancelar: 204 |
| Qualquer erro de validação: 400 (nunca 422) |
| Inexistente ou cancelado: 404 |
| Conflito: 409 |

- Formato do erro: `{"detail": "mensagem"}`
- Precedência quando há mais de um erro: 400 > 404 > 409

## 5. Stack e contêiner

- Python 3.12 + FastAPI; persistência em memória.
- Um único contêiner Docker com a API, na porta 8000 em 0.0.0.0. Não há contêiner de banco.
- Dependências permitidas (lista fechada): [fastapi, uvicorn, pytest, httpx].

## 6. Encapsulamento

- Componentes: Rotas, Modelos, Serviço e Store. Cada um só é usado pela sua interface pública.
- Dependência em um sentido só: Rotas > Serviço > Store.
- Regra de negócio fica só no Serviço. Só o Store toca nos dados.

## 7. Artefatos obrigatórios

Todo código gerado inclui `Dockerfile`, `README.md`, `requirements.txt` e testes.

## 8. Regras de teste

- Um `def test_` por cenário do tests.
- Testes independentes entre si; reset do Store antes de cada um.






