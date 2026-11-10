# ADR 0003: Destinos e quarentena são locais de estoque

- **Status:** aceita
- **Data:** outubro de 2026

## Contexto

O material sai dos almoxarifados para ambulâncias, eventos e domicílios de pacientes. Em uma planilha isso parece uma saída, mas o material continua sendo da empresa até ser usado, e o que sobra precisa voltar. Material devolvido ou rejeitado precisa ser avaliado antes de ser usado de novo. O contrato lista quatro status de lote: disponível, reservado, quarentena e bloqueado.

## Decisão

Destinos e áreas de retenção são locais no mesmo ledger.

- **Ambulância e evento** são locais. Enviar material para eles é uma transferência, um par de movimentos vinculados, nunca uma saída solta. Um evento precisa ser fechado: consumo item por item, o restante devolvido por transferência e qualquer falta registrada como perda, com justificativa. O local do evento é então desativado.
- **Domicílio do paciente** é um local criado automaticamente para cada paciente. Neste módulo, a saída para um paciente é uma transferência para o domicílio seguida de consumo na mesma transação. Quando o próximo módulo registrar o consumo no domicílio, vai usar o mesmo ledger sem reescrevê-lo.
- **Quarentena** é um único local protegido. Devoluções de paciente, recebimentos rejeitados e retenções manuais vão para lá, cada um com o registro do motivo. Só a avaliação do gestor tira material de lá: reintegrar, descartar ou manter retido. A reintegração é recusada para embalagem rompida, lote ilegível ou lote vencido ou bloqueado.
- O único status gravado em um lote é "bloqueado". Disponível, reservado, quarentena e vencido são calculados a partir de saldos e datas.
- **Compras de passagem** (compradas para um paciente e entregues na hora) geram um registro completo sem movimento e sem saldo: rastreabilidade sem estoque fictício.

## Alternativas consideradas

- **Tratar destinos como saídas.** Perde a propriedade do material e torna impossível fechar um evento.
- **Status no lote.** Duas fontes de verdade que discordariam do ledger.

## Consequências

- Um único ledger responde "onde está" para todo item e lote, incluindo o que está em uma ambulância ou no domicílio de um paciente.
- Mais locais para proteger: a Row Level Security filtra por local, então cada perfil vê só os locais de que seu trabalho precisa.
- Eventos precisam de uma etapa explícita de fechamento, que é a regra que o cliente queria.
