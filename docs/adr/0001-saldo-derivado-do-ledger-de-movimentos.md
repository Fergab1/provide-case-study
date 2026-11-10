# ADR 0001: Saldo derivado de um ledger de movimentos insert-only

- **Status:** aceita
- **Data:** outubro de 2026

## Contexto

O estoque vivia em planilhas, onde o saldo é um número que alguém digitou. Em home care isso não basta: o mesmo item chega em lotes com validades diferentes, material já prometido a um paciente não pode ser prometido de novo e a operação precisa saber responder quem recebeu determinado lote. Um saldo que pode ser editado não consegue se explicar.

## Decisão

Os movimentos são a única fonte de verdade, e o saldo nunca é editado.

- A tabela de movimentos é insert-only: sem update e sem delete, para nenhum perfil. Um erro é corrigido por um movimento de estorno que aponta para o original.
- Item mais lote é a unidade de saldo. Dois lotes com validades diferentes são dois saldos, nunca um número só. Itens sem controle de lote recebem um lote implícito.
- Uma tabela de saldos guarda uma linha por item, lote e local, com quantidade física e reservada. Só o banco escreve nela, por triggers, na mesma transação do movimento ou da reserva. O disponível é calculado na leitura: físico menos reservado, e zero quando o lote está bloqueado ou vencido ou quando o local é de quarentena ou manutenção.
- Os saldos são guardados na menor unidade de consumo. A conversão de embalagem (uma caixa com 100) é apresentação, nunca armazenamento. As linhas da nota fiscal mantêm a unidade de compra e uma cópia do fator de conversão naquele momento.
- Preço, descrição e lote são registrados no momento de cada movimento, então uma mudança posterior em um cadastro nunca reescreve o passado.
- Um job noturno recalcula todos os saldos físicos a partir dos movimentos e registra qualquer diferença para o gestor, sem corrigi-la em silêncio. Outro job noturno bloqueia lotes vencidos.

## Alternativas consideradas

- **Uma coluna de saldo editável.** Simples, mas perde o histórico, e o log de auditoria nunca conseguiria explicar um número.
- **Saldo calculado só na leitura (uma soma dos movimentos).** Sempre correto, mas uma reserva precisa travar a linha do lote para que dois pedidos não peguem o mesmo item ao mesmo tempo, e as atualizações em tempo real da plataforma acompanham tabelas, não somas calculadas.

## Consequências

- Todo saldo pode ser rastreado até os movimentos, com autor, data, motivo e lote.
- A tabela de saldos nunca pode divergir do ledger. Os triggers são donos dela, a conciliação noturna a vigia e os testes de banco corrompem um saldo de propósito para provar que a conciliação pega a diferença.
- As correções são visíveis por design: um estorno é um movimento, não uma edição.
