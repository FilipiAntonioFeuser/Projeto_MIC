
# Escape Room — Sistemas Microcontrolados

## Bernardos e Filipi

Projeto final da disciplina **DAS 5332 – Arquitetura e Programação de Sistemas Microcontrolados**.

A proposta é montar um Escape Room usando um Arduino Uno. O jogador precisa resolver uma sequência de enigmas antes que o tempo acabe para conseguir desarmar a bomba. Cada enigma tem seu próprio módulo de código, e a máquina de estados organiza a ordem em que eles aparecem.

A gente separou os códigos por função para facilitar o desenvolvimento e deixar mais claro o que cada parte do projeto faz. Por isso, este repositório reúne os módulos do firmware, e não um programa independente para simplesmente abrir e executar sem integração.

## Como o jogo funciona

Ao iniciar, o jogador passa pelo menu e escolhe uma das duas dificuldades. A partir daí, a máquina de estados controla qual enigma deve ser resolvido e verifica o resultado de cada etapa.

| Dificuldade | Tempo inicial | Ordem dos enigmas |
|---|---:|---|
| Fácil | 90 segundos | NFC → Decode → Fios → Vitória |
| Difícil | 60 segundos | Fios → Potenciômetro → NFC → Decode → Vitória |

Em algumas transições, completar um enigma acrescenta 25 segundos ao tempo disponível. No enigma Decode, uma resposta incorreta desconta 10 segundos. Os erros dos outros enigmas são somados em um contador geral. O limite previsto na máquina de estados é de 3 erros no fácil e 2 no difícil.

Os controles previstos são:

- `1` para iniciar o jogo no menu;
- `1` ou `2` para escolher a dificuldade;
- `#` para confirmar uma resposta no teclado;
- `*` para apagar a entrada;
- `1` para voltar ao menu depois do fim da partida.

## Como os códigos estão organizados

Cada arquivo concentra uma parte do jogo:

| Arquivo | O que faz |
|---|---|
| `Maquina` | Coordena o jogo. Controla os estados, recebe a dificuldade escolhida, define a ordem dos enigmas e verifica as condições de vitória e derrota. Também declara as variáveis globais `vitoria` e `erros`. |
| `Codigo_NFC` | Faz a leitura do cartão pelo RC522 e compara o UID lido com o cartão autorizado (`CB 8F 70 26`). A função `nfc()` sinaliza o resultado para a máquina de estados e mostra a mensagem no LCD. |
| `Codigo_FIOS` | Controla o enigma dos fios. Usa interrupção por mudança de pino para identificar qual entrada foi acionada. O fio ligado a A0 é o correto; A1 e A2 representam erros. |
| `Codigo_POT` | Controla o enigma do potenciômetro. Lê o valor analógico em A3, divide a leitura em oito faixas e pede para o jogador manter o potenciômetro na faixa 5 por um segundo. A faixa 7 representa uma condição de perigo e soma um erro. |
| `Decode` | Gera um número aleatório e sua representação binária. O jogador precisa digitar o número em binário corretamente. Uma resposta errada desconta 10 segundos. |
| `Keypad` | Configura o teclado matricial 4×4 e declara o objeto do LCD I²C, que é compartilhado com os outros módulos. |
| `Senha` | Lê as teclas pressionadas e monta a entrada do jogador. `#` confirma, `*` apaga e as outras teclas são adicionadas à senha. |
| `Timer` | Configura o Timer1 para gerar a referência de tempo do jogo. A função `timer()` atualiza o tempo restante. |

## Componentes e periféricos

O projeto utiliza os seguintes componentes:

- **Arduino Uno (ATmega328P):** placa principal do sistema;
- **RC522:** leitor RFID usado no enigma NFC;
- **teclado matricial 4×4:** entrada de comandos e respostas;
- **LCD 16×2 com I²C:** exibe instruções e resultados dos enigmas;
- **potenciômetro:** entrada analógica do enigma de calibração;
- **fios e LEDs:** formam o enigma dos fios e indicam os resultados;

### Pinagem usada atualmente nos códigos

| Componente | Pinos do Arduino Uno |
|---|---|
| Teclado — linhas | D9, D8, D7 e D6 |
| Teclado — colunas | D5, D4, D3 e D2 |
| LCD I²C | SDA/A4 e SCL/A5, endereço `0x26` |
| RC522 — SS/SDA | D10 |
| RC522 — SPI | SCK/D13, MOSI/D11 e MISO/D12 |
| RC522 — reset | A0 |
| Potenciômetro | A3 |
| Enigma dos fios | A0, A1 e A2 |
| LEDs dos módulos Fios/Potenciômetro | D13 e D12 |

**A pinagem ainda precisa ser ajustada para o uso simultâneo de todos os módulos.** Nos códigos atuais, o RC522 usa D12 e D13 para SPI, mas esses mesmos pinos também aparecem como saídas dos LEDs dos módulos Fios e Potenciômetro. Além disso, A0 está sendo usado tanto pelo reset do RC522 quanto pelo enigma dos fios. Antes da montagem final, é necessário escolher uma pinagem sem conflitos e atualizar os códigos e o circuito.

## Como os módulos se comunicam

A máquina de estados acompanha o resultado dos enigmas por meio de variáveis compartilhadas. As principais são:

- `vitoria`: indica que o enigma atual foi resolvido;
- `erros`: conta os erros cometidos durante a partida;
- `estado`: indica em qual etapa do jogo estamos;
- `dificuldade`: guarda a dificuldade escolhida;
- `tempo`: guarda o tempo restante;
- `senha` e `confirma`: armazenam e controlam a entrada do teclado.

Quando um enigma é resolvido, seu módulo atualiza `vitoria`. Quando o jogador erra, os módulos que trabalham com o contador geral incrementam `erros`. A máquina de estados consulta essas informações para decidir se avança para o próximo enigma ou se encerra a partida.


## Bibliotecas e recursos utilizados

Entre as bibliotecas e recursos presentes nos códigos estão:

- `Keypad`, para o teclado matricial;
- `Wire` e `LiquidCrystal_I2C`, para o display;
- `SPI` e `MFRC522`, para a comunicação com o RC522;
- `avr/io.h` e `avr/interrupt.h`, para acessar os registradores e configurar interrupções do ATmega328P.

