# How to play
####Um guia rápido de implementação e uso. :tent: [:joy:](https://gist.github.com/rxaviers/7360908)

ver [EEPROM Library](https://www.arduino.cc/en/Reference/EEPROM)
ver [TIME](https://playground.arduino.cc/Code/Time/)

J. W. B. de Araújo ¹
F. Ferrari ¹
E. M. Kakuno ²<br>

1 - IMEF, Universidade Federal do Rio Grande - RS.
2 - Unipampa, campos Bagé - RS.<br><br>

A Mini CAN é um produto de mestrado do [Programa de Pós-Graduação em Física](https://ppgfisica.furg.br/), IMEF, FURG, e foi parcialmente custeado pela [CAPES](https://www.gov.br/capes/pt-br) através de uma bolsa.

### Introdução
<div style="text-align: justify"> <font size="3pt" style="arial">
A Mini CAN é um projeto de rede de sensores CAN para monitoramento, seus principais objetivos (ainda que não pareça) são: Acessibilidade; Confiança; Modularidade.
Este guia segue a filosofia FIsso-OIsso (Faça Isso, Obtenha Isso). O guia possui duas configurações, no primeiro é descrito a menor implementação possível, e no segundo uma variante da mesma. Ao fim se tem um conjunto de dicas e uma listagem referênciais para mais detalhes.
<div>

## Configuração A - layout mínimo.
<div style="text-align: justify"> <font size="3pt" style="arial">
O layout mínimo é composto de um nodo sensor (CAN Sensor), de um nodo monitor (CAN Mon) e de um computador, com python, py serial etc.
<div>

#### Lista de materiais:
- [ ] 1 Computador com linux, Arduino IDE, python, pySerial, demais bibliotecas.
- [ ] 2 Arduino Nano.
- [ ] 2 Módulo CAN (MCP2515 + TJA1050).
- [ ] 1 par trançado para dados (CAN High).
- [ ] 2 fios para alimentação.
- [ ] Fios para conexões entre os Arduino e os módulos CAN.

### Software
#### Instalação da da biblioteca MCP2515:
1. Baixe a versão estável (ramo master), disponivel em https://github.com/KakiArduino/MCP2515.

2. No ambiente Arduino IDE, inclua a bibblioteca selecionando o arquivo ***MCP2515-master.zip*** pelo caminho: ***Sketch/Incluir Biblioteca/Adicionar Biblioteca .ZIP***

2. Adicione o arquivo  ***MCP2515-master.zip*** na pasta de bibliotecas do Arduino. Em caso de duvidas sobre instalação de biblioteca por esse procedimento consulte: [arduino.cc/en/Guide/Libraries](https://www.arduino.cc/en/Guide/Libraries#importing-a-zip-library).

#### Programando os nodos CAN Mon e CAN Sensor:
1. No ambiente Arduino IDE, importe o exemplo CANMon, disponível em ***Arquivo/Exemplos/MCP2515-master/CANMon***.

2. Selecione a tipo de placa em ***Ferramentas/Placas***. Algumas placas são comercializadas com mais de um tipo de processadores ou bootloader, po isso verifique se opção pré-selecionada é a correta em ***Ferramentas/Processador***, caso não seja erros de gravação podem ocorrer.

3. Conecte uma das placas Arduino e selecione a porta serial atribuida à ela em, *Ferramentas/Porta*, após isso carregue o código fonte na placa apertando o botão ***Carregar*** ou teclando ***Ctrl + u***.

> Ob. 1: A comunicação serial do CAN Mon é por padrão 2000000, porém nem todas interfaces de monitoramento serial possuem essa velocidade, verifique! Caso necessario modifique a linha `Serial.begin(2000000);` na `void loop(){...}`.

4. Repita os procedimentos *2.1, 2.2 e 2.3* porém agora com o exemplo CANSensor, disponível em *Arquivo/Exemplos/MCP2515-master/CANSensor*, e gravando na outra placa Arduino Nano.

5. (Opicional) Para alterar a informação enviada pelo nodo CAN Sensor, deve-se atribuir o valor desejado a variavel leitura, alterando a linha `leitura.inte = analogRead(A0)` na `void lopp(){...}`. Os tipos de variaveis pré-configurados para envio são: long int (`leitura.inte`); unsigned long int (`leitura.uinte`); float (`leitura.flt`) ;  int64_t (`leitura.int64`); uint64_t (`leitura.uint64`); int32_t (`leitura.int32`); uint32_t (`leitura.uint32`); int16_t (`leitura.int16`); uint16_t (`leitura.uint16`). Você pode optar por enviar uma lista de bytes, com até 8 bytes, para isso forneça o tamanho da lista em bytes no terceiro paramentro e o nome da lista no quarto paramentro da função `putinBuff(ID_padrao, ID_extecao, num_bytes, data_to_send)`.

> Ob. 2: Ao substituir a placa Arduino Nano por outro módelo pode ser necessário alterar o pino digital ultilizado na interrupção, que é declarada na ultima linha da `void setup(){...}`. Em [arduino.cc/reference/](https://www.arduino.cc/reference/pt/language/functions/external-interrupts/attachinterrupt/) pode-se verificar os pinos de entrada de interrupções para os variados tipos de placas Arduino.

### Conexões
![CANMon_CANSensor](/mon_sensor_cx.jpg)
> Ob. 3: Repare que o jumper do terminador está fechado em ambos os nodos. Se forem adicioandos outros nodos sensores no barramento os teminadores devem ser ligados apenas no dois nodos extremos do barramento.

### Coletando as informações no computador.

Para coletar as informações da rede deve-se conectar o nodo CAN Mon a um computador e rodar o scrritp ***CANMon.py***, localizado no caminho ***.../Arduino/libraries/MCP2515-master/extras/CANMon.py***. será solicitado a porta serial na qual o Arduino está conectado, um endereço e nome para salvar o arquivo de dados.

## Configuração B - layout autonomo mínimo.

O processo de programação dos nodos é muito semelhante a feita na configuração A, porém lo lugar de usar o exemplo CANMon.ino, será usado o CANSave.ino. Para o CANSensor nada muda
