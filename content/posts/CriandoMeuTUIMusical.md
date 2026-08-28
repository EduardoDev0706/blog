+++
date = '2026-07-23T14:38:12-03:00'
draft = false
title = 'Fazendo um TUI Musical'
summary = 'Minha experiência criando um aplicativo desktop de música que roda no terminal'
+++

## A Ideia inicial

Eu gosto bastante de manter meus aparelhos eletrônicos limpos e organizados, e isso obviamente é regra para o meu celular, principalmente quando falamos da galeria. Acabo por tirar um monte de prints de coisas que eu acho interessante, seja em fóruns ou nos posts do Instagram. Em um destes prints estava o link de um site que eu fiquei interessado, chamado [mynoise](https://mynoise.net/). A proposta do site é basicamente ser um soundscape, com vários tipos de som ambiente autorais, diga-se de passagem muito bons, adoro a seleção que o site possui. Por meio desse agrado, eu pensei se eles teriam uma API, para criar um software que usasse a base deles, mas com a minha cara/necessidades. No entanto está não é a proposta do site, e eles deixam bem claro que é proíbido usufruir de qualquer uma das produções fora do ambiente deles. OK! Vou buscar saber como fazer o meu próprio.

## Partindo pros requisitos

Outro tipo de coisa que eu gosto bastante são TUIs, são softwares mais leves que rodam no terminal com algum recurso gráfico mais limpo. Decidi unir esse gosto por sons de ambiente com um TUI, e produzir algo com isso. Abaixo fica o desenho da minha definição de requisitos para esse projeto, no qual optei por usar Node.js pro backend.

![Quadro de Requisitos](/img/posts/diagrama.png)

## As etapas do projeto

O projeto foi dividido em 5 etapas, utilizei a IA pra me auxiliar nesse passo a passo.

- [Etapa 1] -> Prototipar reprodução de áudio via script sem interface
- [Etapa 2] -> Criar o layout estático no TUI (desenhar os quadros)
- [Etapa 3] -> Conectar os eventos de teclado (pressionar 'Espaço' chama Play/Pause)
- [Etapa 4] -> Consumir a API para carregar a lista de áudios dinamicamente
- [Etapa 5] -> Sincronizar a barra de progresso do TUI com o tempo real do áudio

Tendo isso em mente, podemos passar para como cada etapa foi realizada.

### Etapa 1 - Prototipando a Reprodução do áudio

Um dos problemas de utilizar o Node.js é que ele não possui um motor de áudio embutido, mas existem opções bem competentes nesse caso (e outras nem tanto). Inicialmente eu tentei utilizar os packages 'Speaker' e 'Wav', no qual fazem bindings com bibliotecas do C++. Mas finalizado o protótipo, o áudio não saia de jeito nenhum, então fiz alguns testes.

- 1°: Verifiquei se era um problema de error_handling, não era o caso.
- 2°: Analisei a lógica do código e vi se existia algum processo que estava matando o áudio antes de rodar fisicamente na placa, também não.
- 3°: Testar com uma ferramenta de fora do código, rodou. Quer dizer que o arquivo .wav que eu estava usando não estava corrompido, e o comando para o meu SO gerava o resultado.

O que deu pra concluir é que esses pacotes estavam obsoletos, então parti para uma outra solução: child_process, um módulo nativo do Node.js. Em vez de terceirizar a solução para uma biblioteca externa, optei por usar as ferramentas que eu já possuia no meu SO e usar um motor de áudio real, no caso o aplay. FOI! RODOU! E em menos linhas de código ;D.

Com isto concluído, podemos seguir em frente.

### Etapa 2 - Hora de desenhar

Para desenhar as caixas e todo o resto do TUI, utilizei o 'blessed', um package do nosso querido npm.

Desenhei o esqueleto do TUI em um novo arquivo, usando como base a referência que está na imagem dos requisitos, ficou assim:
![Imagem do TUI](/img/posts/playerTUI.png)

### Etapa 3 - Conectando o TUI com o áudio

É aqui que nasce o State Manager. No caso dessa aplicação, ele vai ficar responsável por transferir os dados do nosso TUI para o motor de áudio.
