# Instruções — Zona Azul Digital

## 0. Ordem de leitura
1 constituição
2 spec
3 plan
4 testes
5 tasks

## 1. Papel

O seu papel será mplementar exatamente o que está em constituição, spec, plan, tests e tasks. Em conflito vale: constituição > spec > plan > testes > tasks

## 2. Fluxo

1. Uma task por vez, na ordem do tasks.
2. Em cada task: escrever os testes dela, ver falhar, implementar, rodar a suíte inteira.
3. Só passar para a próxima com tudo ok.


## 3. Nunca

- Inventar rota, campo, status code ou regra que não esteja no spec.
- Apagar ou burlar um teste para ele passar.
- Usar dependência fora da lista da constituição.
- Furar o encapsulamento: o sentido é Rotas > Serviço > Store, e regra de negócio fica só no Serviço.

## 4. Em caso de dúvida

Seguir a ordem de prioridade. Nunca tente adivinhar. (Se nenhum arquivo cobrir o caso, escolha a interpretação mais simples que não contradiga nenhum deles, sem criar rota ou campo novo, e registre a decisão no README.md. Nunca pare para perguntar: ninguém vai responder)

## 5. Formato da resposta

Arquivos completos, cada um com o seu caminho. Nada de trechos soltos.
