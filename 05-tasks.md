# Tasks — Zona Azul Digital

Ordem de execução. Toda task de UC é "escrever os testes do UC, depois implementar".

| ID | Descrição | Depende de | UC | Componente | Testes que passam |
|---|---|---|---|---|---|
| TK1 | Scaffolding: arquivos do plan, `config.py` com as 5 constantes, `clock.py`, app, handlers de erro `{"erro": "<codigo>"}`, Store com reset, fixture de teste | — | — | todos | — |
| TK2 | Modelos: bilhete (campos e status) e erro de domínio (código + status HTTP) | TK1 | — | Modelos | — |
| TK3 | Abrir bilhete: validação de `placa` e `entrada`, criação com ID sequencial | TK2 | UC1 | Serviço, Rotas, Store | cenários do UC1 |
| TK4 | Encerrar bilhete: cálculo de `minutos`, tolerância, frações e teto | TK3 | UC2, UC7 | Serviço, Rotas | cenários do UC2 e UC7 |
| TK5 | Cancelar bilhete | TK3 | UC5 | Serviço, Rotas | cenários do UC5 |
| TK6 | Uma vaga por placa: 409 com bilhete aberto; libera após encerrar ou cancelar | TK4, TK5 | UC8 | Serviço, Store | cenários do UC8 |
| TK7 | Listar ativos, com ordenação | TK5 | UC3 | Serviço, Rotas | cenários do UC3 |
| TK8 | Histórico por placa | TK5 | UC6 | Serviço, Rotas | cenários do UC6 |
| TK9 | Relatório diário: total, faturamento e tempo médio | TK4 | UC4 | Serviço, Rotas | cenários do UC4 |
| TK10 | Infraestrutura: `requirements.txt`, `Dockerfile`, `README.md` | TK1 | — | — | — |
| TK11 | Verificação final: `pytest` todo verde; serviço sobe na porta 8003; cada rota do contrato responde; nenhuma resposta com `detail` nem número decimal | todas | todos | — | todos |