# ADR 0004: PWA instalável sobre a biblioteca de componentes da equipe

- **Status:** aceita
- **Data:** outubro de 2026

## Contexto

A primeira versão do app foi construída com Expo e React Native Web, com uma biblioteca de componentes própria feita a partir do arquivo do Figma. Ao mesmo tempo, um colega entregou uma biblioteca de componentes web (React 19, Vite, styled-components) a partir do mesmo arquivo do Figma, validada pixel a pixel em todas as variantes desenhadas. Os fluxos de celular do próximo módulo ainda precisavam ser decididos: app de loja ou app web.

## Decisão

O alvo mobile roda como app web instalável (PWA) no navegador do celular, não como app de loja. O app passou para a biblioteca de componentes da equipe.

- As camadas de dados, sessão e utilitários foram portadas como estavam: 14 dos 16 módulos de dados não precisaram de nenhuma mudança.
- O que era específico do React Native foi trocado por equivalentes web: a fila offline foi para o IndexedDB, o roteamento passou para um hash router para o app rodar em hospedagem estática e o seletor de fotos virou um input de arquivo com redimensionamento no client.
- A biblioteca da equipe não é editada. As peças que ela não tem (erro de campo, botão em loading, preview de upload) ficam em uma pasta separada, derivadas dos tokens dela, para que as verificações de fidelidade continuem válidas.
- O app antigo só foi removido depois que o novo foi revisado.

## Alternativas consideradas

- **Manter Expo e React Native Web.** Duas bibliotecas de componentes para o mesmo design, além de peculiaridades da web, como stacking contexts que escondiam dropdowns abertos atrás do campo seguinte.
- **Um app de loja.** Revisão e distribuição pela loja para uma equipe interna pequena, sem nenhuma funcionalidade que um PWA não cobrisse neste módulo.

## Consequências

- Uma biblioteca de componentes para toda a equipe e uma stack para desktop e celular.
- A suíte de testes do app foi reescrita de React Native Testing Library para React Testing Library e Vitest, o maior bloco individual da migração.
- Os limites do PWA no iOS (câmera, background sync, modo standalone) são um risco conhecido para o próximo módulo. O modo standalone no iOS já precisou de meta tags próprias, e os fluxos de câmera serão validados cedo em um iPhone real.
