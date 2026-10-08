# Spec — — Zona Azul Digital

## 0. Contexto
Operadora de estacionamento rotativo precisa de uma API para abrir e encerrar bilhetes por placa, listar bilhetes ativos e emitir relatório diário. O back-office não existe — somente a API é objeto desta prova.

## 1. Casos de uso

URL base:http://localhost:{PORTA_SERVICO}

### UC1 — Abrir bilhete
POST /bilhetes— corpo {"placa": "ABC1D23"}(7 caracteres alfanuméricos, maiúsculos) → 201 {"id": 1, "placa": "ABC1D23", "entrada": "<ISO-8601 com fuso -03:00>", "status": "aberto"}

O corpo aceita entradaopcional (ISO-8601 com fuso): quando presente, o bilhete abre naquele instante em vez de "agora". É o gancho de testabilidade da correção — sem ele, testar fração/teto exigia esperar tempo real. Formato inválido → 422 {"erro": "entrada_invalida"} .


### UC2 — Fechar bilhete
POST /bilhetes/{id}/encerramento→ 200 :

{"id": 1, "placa": "ABC1D23", "entrada": "...", "saida": "...",
 "minutos": 95, "valor_centavos": 1250}
boa quantia de valor:

Cobra-se por fração de FRACAO_MINUTOSminutos, arredondando para cima (fração exata cobra 1 fração; 1 minuto a mais já seguinte cobra a fração);
hora cheia = TARIFA_HORA_CENTAVOS; valor da fração = tarifa ÷ (60 ÷ FRACAO_MINUTOS);
aplica-se o teto diário : valor_centavosnunca supera TETO_DIARIO_CENTAVOS;
valor sempre em centavos, inteiro — a API nunca retorna ponto flutuante. 


### UC3 — Listar ativo
GET /bilhetes/ativos→ 200 com array dos bilhetes abertos, mais recentes primeiro.


### UC4 — Relatório diário
GET /relatorios/diario?data=AAAA-MM-DD→ 200 :

{"data": "2026-10-05", "total_bilhetes": 12,
 "faturamento_centavos": 8400, "tempo_medio_minutos": 47}

### UC5 — Cancelar bilhete
POST /bilhetes/{id}/cancelamento→ 200 com status: "cancelado". Só bilhetes abertos podem ser cancelados — sem cobrança (não gera saida nem valor_centavos).

### UC6 — Histórico por placa
GET /bilhetes?placa=ABC1D23→ 200 com matriz de todos os bilhetes da placa (qualquer status), mais recentes primeiro. Placa que nunca foi colocada → array vazio.

### UC7 — Tolerância gratuita
Os primeiros TOLERANCIA_MINUTOSde um bilhete são grátis : duração ≤ tolerância → valor_centavos: 0. Passou da tolerância (mesmo por 1 minuto) → cobra integral desde o primeiro minuto — a tolerância não é descontada.

### UC8 — Uma vaga por placa
POST /bilhetespara placa que já tem bilhete aberto → 409 {"erro": "bilhete_em_aberto"} . Após encerrar ou cancelar, a placa volta ao poder de abertura.


### Erros
Situação	Status	Corpo
Placa de mercado ou inválida	422	{"erro": "placa_invalida"}
entradafora de ISO-8601	422	{"erro": "entrada_invalida"}
datafora deAAAA-MM-DD	422	{"erro": "data_invalida"}
Bilhete inexistente	404	{"erro": "bilhete_nao_encontrado"}
Fechar bilhete já encerrado	409	{"erro": "bilhete_ja_encerrado"}
Cancelar bilhete não aberto	409	{"erro": "bilhete_nao_aberto"}
Abrir bilhete com placa já ocupada	409	{"erro": "bilhete_em_aberto"}