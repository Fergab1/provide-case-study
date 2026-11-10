# ADR 0002: Regras de estoque ficam em funções do banco

- **Status:** aceita
- **Data:** outubro de 2026

## Contexto

As mesmas regras precisam valer no desktop do almoxarifado, no celular em campo e em uma fila offline que reenvia o trabalho depois. A maioria das operações de estoque mexe em várias linhas de uma vez (movimento, saldo, reserva, entrada de auditoria) e precisa dar certo ou falhar por inteiro. Cinco perfis têm direitos diferentes, e esses direitos precisam valer mesmo que um client se comporte mal.

## Decisão

Toda operação de estoque é uma função SQL chamada por RPC: recebimento contra nota fiscal, saída, transferências, devoluções de paciente, descartes, avaliação de quarentena, fechamento de evento, contagens cegas, ajustes de inventário e estornos.

- O app nunca escreve diretamente nas tabelas de estoque. A Row Level Security bloqueia isso, e as funções verificam internamente o perfil de quem chama.
- Cada chamada carrega uma chave de idempotência gerada no dispositivo. Repetir uma chamada com a mesma chave devolve o resultado original sem escrever de novo, então a fila offline pode reenviar com segurança.
- As funções falham com erros codificados. O app traduz cada código em uma mensagem, e um teste lê as migrations e falha se algum código não tiver tradução.
- Edge Functions são usadas só onde é preciso um segredo (criar usuários com senha temporária e enviar o e-mail de acesso). Trabalho pesado que não precisa de segredo, como ler planilhas, roda no dispositivo.

## Alternativas consideradas

- **Um servidor de API separado.** Mais um serviço para hospedar, proteger e pagar, com um único desenvolvedor, para fazer o que o banco faz em uma transação.
- **Regras no app, só com RLS.** A RLS consegue dizer quem pode escrever uma linha, mas não garante um invariante de várias linhas, e as retentativas da fila offline duplicariam trabalho.

## Consequências

- Cada operação é uma transação, com as mesmas regras para qualquer client.
- A maior parte da lógica de negócio é SQL, então a suíte de testes do banco (pgTAP) carrega a maior parte do peso: toda função, toda recusa e toda permissão têm asserções.
