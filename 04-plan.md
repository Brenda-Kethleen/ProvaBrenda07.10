# Plan  — Zona Azul Digital

## 1. Stack

| Item | Escolha | Por quê |
|---|---|---|
| Linguagem | [Python 3.12] | [exigida pelo enunciado] |
| Framework | [FastAPI] | [validação e rotas prontas] |
| Persistência | em memória | [enunciado não pede banco] |
| Testes | [pytest + httpx] | [TestClient do FastAPI] |

## 2. Arquivos, componentes e encapsulamento

| Arquivo | Componente do spec | Responsabilidade | Pode importar |
|---|---|---|---|
| `main.py` | Rotas | endpoints e handler de erro | `service`, `models` |
| `models.py` | Modelos | modelos de entrada e saída | — |
| `service.py` | Serviço | regras de negócio | `store`, `models` |
| `store.py` | Store | dados em memória, IDs e reset | — |
| `test_app.py` | testes | um `def test_` por cenário | `main`, e `store` só para o reset |
| `requirements.txt` | — | dependências | — |
| `Dockerfile` | contêiner api | imagem da API | — |
| `README.md` | — | como rodar e testar | — |

## 3. Decisões técnicas

### Erros
Situação	Status	Corpo
Placa de mercado ou inválida	422	{"erro": "placa_invalida"}
entradafora de ISO-8601	422	{"erro": "entrada_invalida"}
datafora deAAAA-MM-DD	422	{"erro": "data_invalida"}
Bilhete inexistente	404	{"erro": "bilhete_nao_encontrado"}
Fechar bilhete já encerrado	409	{"erro": "bilhete_ja_encerrado"}
Cancelar bilhete não aberto	409	{"erro": "bilhete_nao_aberto"}
Abrir bilhete com placa já ocupada	409	{"erro": "bilhete_em_aberto"}

- Erro de validação: handler que troca o 422 do framework por 400 com `{"detail": "mensagem"}`.
- ID: contador sequencial a partir de 1, dentro do Store.
- [Sobreposição: conflita se `inicioA < fimB` e `inicioB < fimA`; adjacente não conflita.]
- [Arredondamento: `ceil`, sempre para cima.]
- [Cancelamento: muda o status e mantém o registro (soft delete); cancelado responde 404 e some das listagens.]
- [Datas: ISO 8601 sem fuso; "agora" é o horário local do servidor.]
- Testes: fixture que chama o reset do Store antes de cada teste.

## 4. requirements.txt

fastapi
uvicorn
pytest
httpx

## 5. Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 6. README.md deve conter

- Descrição do sistema e lista de endpoints.
- Docker: `docker build -t app .` e `docker run -p 8000:8000 app`
- Podman: `podman build -t app .` e `podman run -p 8000:8000 app`
- Local: `pip install -r requirements.txt` e `uvicorn main:app --host 0.0.0.0 --port 8000`
- Testes: `pytest`
# Plan — Zona Azul Digital

## 1. Stack

| Item | Escolha | Por quê |
|---|---|---|
| Linguagem | Python 3.12 | `datetime.fromisoformat` aceita ISO-8601 com offset sem biblioteca extra |
| Framework | FastAPI + uvicorn | rotas e JSON prontos, sobe com um comando |
| Persistência | em memória | o escopo é só a API; não há requisito de durabilidade |
| Testes | pytest + httpx | `TestClient` do FastAPI roda sem subir servidor |

## 2. Arquivos, componentes e encapsulamento

| Arquivo | Componente | Responsabilidade | Pode importar |
|---|---|---|---|
| `main.py` | Rotas | endpoints, leitura do corpo, handlers de erro | `service`, `models` |
| `models.py` | Modelos | bilhete e erro de domínio (código + status) | — |
| `service.py` | Serviço | validações e regras de negócio | `store`, `models`, `config`, `clock` |
| `store.py` | Store | dados em memória, contador de IDs e reset | `models` |
| `config.py` | — | as 5 constantes da variante | — |
| `clock.py` | — | função `now()`, único ponto de leitura do relógio | — |
| `test_app.py` | testes | um `def test_` por cenário do tests.md | `main`, `clock`, e `store` só para o reset |
| `requirements.txt` | — | dependências | — |
| `Dockerfile` | — | imagem da API | — |
| `README.md` | — | como rodar e testar; seção "Decisões" | — |

## 3. Decisões técnicas

- **Dinheiro:** centavos inteiros; float acumula erro (0.1 + 0.2 ≠ 0.3).
- **Arredondamento:** só aritmética inteira. Frações: `-(-minutos // 15)`. Tempo médio: `(2 * soma + n) // (2 * n)`. Proibido `round()` (half-even).
- **Erros:** o Serviço lança erro de domínio; handler em `main.py` responde `{"erro": "<codigo>"}`. Handlers padrão do FastAPI sobrescritos: nunca sai `{"detail": ...}`.
- **Validação:** manual, no Serviço. Rotas recebem corpo como JSON cru e `id`, `placa`, `data` como string, para o FastAPI não gerar 422 próprio.
- **Relógio:** `clock.now()` em offset fixo `-03:00`, sem microssegundos; substituído nos testes, sem `sleep`.
- **Datas:** entrada lida com `datetime.fromisoformat`, convertida para `-03:00`; saída com `isoformat()`.
- **ID:** contador sequencial a partir de 1, no Store.
- **Cancelamento:** só muda o `status`; o registro permanece.
- **Testes:** fixture que reseta o Store antes de cada teste.

## 4. requirements.txt

```
fastapi
uvicorn
pytest
httpx
```

## 5. Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8003
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8003"]
```

## 6. README.md deve conter

- Descrição do sistema e lista de endpoints.
- Local: `pip install -r requirements.txt` e `uvicorn main:app --host 0.0.0.0 --port 8003`
- Docker: `docker build -t app .` e `docker run -p 8003:8003 app`
- Testes: `pytest`
- Seção "Decisões" com qualquer interpretação adotada.