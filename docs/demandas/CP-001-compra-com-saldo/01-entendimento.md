# CP-001 — Compra com saldo · 01 Entendimento

> Etapa: **01 Entender**. Este documento não implementa nada e não propõe solução técnica.
> Ele separa o que é **fato**, **lacuna**, **pergunta**, **hipótese** e **decisão necessária**.

## Demanda recebida

> "Permitir que o usuário utilize o saldo do cartão para adquirir produtos disponíveis na plataforma."
> — `docs/backlog.md`, seção "Utilizar saldo em produtos"

O próprio backlog diz que a demanda está "registrada sem regras extras" e que não deve ser implementada "até ser pedida com critérios claros".

## 1. Fatos conhecidos

### Da demanda
| # | Fato | Fonte |
|---|---|---|
| F1 | O ator é "o usuário". | `docs/backlog.md` |
| F2 | O meio de pagamento é o saldo do cartão. | `docs/backlog.md` |
| F3 | O objeto da operação são "produtos disponíveis na plataforma". | `docs/backlog.md` |

### Do sistema atual
| # | Fato | Fonte |
|---|---|---|
| F4 | A compra ainda não existe: não há endpoint, e o botão "Usar cartão" aparece desabilitado com "Em breve". | `README.md` (Endpoints), `mobile/src/componentes/CardProduto.tsx` |
| F5 | O cartão tem `Saldo` (`decimal`), que começa em zero e hoje só **aumenta** (por recarga). | `Cartao.cs` |
| F6 | Não existe regra no domínio sobre saldo negativo, porque hoje nenhuma operação diminui o saldo. | `Cartao.cs` |
| F7 | A movimentação tem só `CartaoId`, `Valor`, `Descricao` e `DataHora`. Não tem tipo (entrada/saída) nem referência a produto. | `MovimentacaoCartao.cs` |
| F8 | O histórico no app é apresentado como "Histórico simples das recargas" e mostra todo valor com sinal "+". | `mobile/src/telas/TelaMovimentacoes.tsx` |
| F9 | O produto tem `Preco` (`decimal`) e um campo `Disponivel` (`bool`). Os 6 produtos do seed estão com `Disponivel = true`. | `Produto.cs`, `ProdutoConfiguracao.cs` |
| F10 | O catálogo vem de seed, sem administração, estoque nem carrinho. | `docs/product.md`, `docs/architecture.md` |
| F11 | Não há login. O app guarda o cartão selecionado localmente, e a API pode ter vários cartões. | `docs/product.md` |
| F12 | Há um precedente de consistência: a recarga grava saldo e movimentação no mesmo `SaveChanges`. | `AGENTS.md`, `docs/fluxo-recarga.md` |
| F13 | Hoje a orientação é não implementar compra, estoque, carrinho, checkout, autenticação ou integração bancária. | `AGENTS.md` |

## 2. Lacunas — o que a demanda não diz

- **Quando a compra é permitida:** saldo insuficiente, produto indisponível ou cartão inexistente.
- **Como o saldo muda:** o valor debitado, se o saldo pode ficar negativo e se aceita pagamento parcial.
- **Quantidade:** uma unidade, várias unidades, vários produtos de uma vez.
- **Registro:** como a compra aparece no histórico e que informações guarda.
- **O que é "produto disponível":** o campo `Disponivel`, estoque ou outra regra.
- **Experiência do usuário:** se há etapa de confirmação, o que ele vê antes e depois.
- **Erros:** que mensagens existem e o que acontece se algo falhar no meio da operação.
- **Depois da compra:** se existe cancelamento, estorno ou algum comprovante.

## 3. Perguntas abertas (para o responsável pelo negócio)

**Usuário e objetivo**
1. Quem compra é sempre o titular do cartão selecionado no app? Sem login, basta ter o cartão selecionado?
2. Qual resultado o usuário espera ver ao final: saldo atualizado, mensagem, registro no histórico?

**Regras**

3. O valor debitado é exatamente o preço do catálogo no momento da compra?
4. Cada compra é de uma unidade de um único produto, ou pode haver quantidade ou vários produtos?
5. O usuário precisa confirmar a compra depois de ver o produto e o valor?

**Restrições**

6. A compra é bloqueada quando o saldo é menor que o preço? O saldo pode ficar negativo em algum caso?
7. "Produto disponível" significa `Disponivel = true`? Produto indisponível aparece no catálogo, aparece mas bloqueado, ou some?
8. Existe limite de valor ou de número de compras?

**Exceções**

9. Quais erros o usuário deve ver: saldo insuficiente, produto indisponível, cartão ou produto inexistente, falha técnica?
10. Se o débito acontecer e o registro falhar (ou o contrário), o que deve acontecer? A operação precisa ser "tudo ou nada"?
11. Duas compras simultâneas que juntas passem do saldo: como tratar?

**Histórico**

12. A compra entra no mesmo histórico das recargas? Precisa mostrar produto, quantidade, valor, data e hora?
13. Como diferenciar visualmente compra de recarga (sinal, cor, texto)?

**Escopo**

14. Cancelamento, estorno, carrinho e múltiplas unidades estão dentro ou fora desta entrega?

## 4. Hipóteses (podem ser verdade, mas **não são regra**)

| # | Hipótese | Por que parece plausível | Confirmar com |
|---|---|---|---|
| H1 | O saldo é debitado pelo preço do catálogo. | É o único preço que existe (F9). | Pergunta 3 |
| H2 | "Disponível" corresponde ao campo `Disponivel`. | O campo já existe no modelo (F9). | Pergunta 7 |
| H3 | A compra é bloqueada com saldo insuficiente, e o saldo nunca fica negativo. | Cartão lúdico, sem crédito (`docs/product.md`). | Pergunta 6 |
| H4 | Uma compra é uma unidade de um único produto. | Não há carrinho nem estoque (F10, F13). | Pergunta 4 |
| H5 | A compra aparece no mesmo histórico, como saída. | O histórico é o único registro de movimentação (F7, F8). | Perguntas 12 e 13 |
| H6 | Débito e registro precisam acontecer juntos. | Precedente da recarga (F12). | Pergunta 10 |

## 5. Decisões necessárias (quem é responsável precisa escolher)

| # | Decisão | Quem decide |
|---|---|---|
| D1 | Regra de permissão da compra: saldo suficiente, produto disponível, cartão existente. | Negócio / PO |
| D2 | Quantidade e escopo: uma unidade, um produto; carrinho, cancelamento e estorno dentro ou fora. | Negócio / PO |
| D3 | Significado de "produto disponível". | Negócio / PO |
| D4 | Como a compra é registrada e exibida no histórico (dados e diferenciação de recarga). | Negócio / PO + UX |
| D5 | Lista de erros e mensagens ao usuário. | Negócio / PO + UX |
| D6 | Garantia de consistência (tudo ou nada) diante de falha. | Negócio define a regra; técnico garante |
| D7 | Liberar a orientação do `AGENTS.md` que hoje proíbe implementar compra. | Responsável pelo projeto |

## 6. Próximo passo

Levar as perguntas e decisões ao responsável e transformar as respostas em regras e critérios de aceite (`02-historia-rica.md` e `03-criterios-de-aceite.md`). Até lá, **nada de código**.
