# MCP2515 (em contrução)

 >John Welvins Barros de Araujo¹ <br /> 
 >Prof. Fabricio Ferrari¹ <br /> 
 >Prof. Edson M. Kakuno² <br /> 

¹ [IMEF](https://imef.furg.br/) - Universidade Federal do Rio Grande <br /> 
² [Física](http://cursos.unipampa.edu.br/cursos/licenciaturaemfisica/) - Unipampa Bagé <br /> 


A biblioteca MCP2515 foi feita para auxiliar/facilitar o controle do CI da Microchip, de mesmo nome, por plataformas Arduino, ou compatíveis. As funções de configuração possuem valores padrão, de modo a fornecer uma configuração simples e rápida. O CI MCP2515 configurado no modo padrão dessa biblioteca estará preparado para operar em uma rede de 125 k bit/s, sem a implementação de filtros de aceite nos buffers de entrada, com rollover do buffer de saída RXB0 para o RXB1, e além disso são habilitadas interrupções que indicam o sucesso de recebimento e envio. Para setar outra configuração o usuário deverá atualizar os valores das [variáveis de configuração](#varpubs_conf), e em seguida executar a função correspondente aos registros modificados, como a [confMode()](#confMode), [confRX()](#confRX), [confTX()](#confTX), [confINT()](#confINT), [confFM()](#confFM) e [confCAN()](#confCAN). Ou ainda as usando as funções de escrita básica, [write (REG, VAL, CHECK = 1)](#write) e [bitModify (REG, MASK, VAL, check = 0)](#bitModify).



Os frames podem ser escritos no barramento com a função [writeFrame(frameToSend, txb\_ = 0, timeOut = 10, check = 0)](#writeFrame) e podem ser lidos pela função [readFrame ()](#readFrame) que atualiza os valores dos frames internos [frameRXB0](#frameRXB0) e [frameRXB1](#frameRXB1).
As informações da mensagem são armazenadas na estrutura apresentada na figura [Frame](#frame), e podem ser acessados com as [variáveis do frame](#frames_var).
Também é possível obter os dados acessando diretamente os registros correspondentes aos buffers de entrada do MCP2515 com a função [read(REG, data, n = 1)](#read).
De forma análoga pode-se escrever nos buffers de saída do MCP2515 e solicitar o envio usando as funções de escrita básicas.


<div id='frame'/>
<p align="center">
<img src="FURGCAN_frame.png" width="700">
</p>



A principal referência desta biblioteca é o datasheet [MCP2515 Stand-Alone CAN Controller with SPI Interface](http://ww1.microchip.com/downloads/en/DeviceDoc/MCP2515-Stand-Alone-CAN-Controller-with-SPI-20001801J.pdf).
Por questões de comodidade muitos dos códigos numéricos relacionados a comunicação do MCP2515 com a plataforma Arduino foram expressos em hexadecimal indicado por 0x, e quando não estarão em decimal.



Um exemplo de conexão com um Arduino Nano e um módulo CAN, o MCP2515 e o transceptor [TJA1050](https://www.nxp.com/docs/en/data-sheet/TJA1050.pdf) da Phillips, pode ser visto na figura [Conexão](#MCPxNano).


<div id='MCPxNano'/> 
<p align="center">
<img src="example/CANMon/CANMon_cx.png" width="700">
</p>



O barramento é composto por pelo menos dois dispositivos, na figura [Bus](#bus) pode-se ver um diagrama simplificado de um barramento CAN.


<div id='bus'/>
<p align="center">
<img src="600_FURGCAN_barramento.png" width="700">
</p>


São fornecidos 6 exemplos de rotinas, dois para transmissão [CANTX.ino](https://github.com/KakiArduino/MCP2515/blob/version_1/example/CANTX/CANTX.ino) e [CANTXshort](https://github.com/KakiArduino/MCP2515/blob/version_1/example/CANTXshort/CANTXshort.ino), um para recepção [CANRX.ino](https://github.com/KakiArduino/MCP2515/blob/version_1/example/CANRX/CANRX.ino), dois exemplos de nodos sink (receptores), o [CANMon.ino](https://github.com/KakiArduino/MCP2515/tree/version_1/example/CANMon) envia para as mensagens para um computador pela porta USB-Serial, e o [CANSave.ino](https://github.com/KakiArduino/MCP2515/tree/version_1/example/CANSave) que salvas as mensagens recebidas em um cartão SD usando SPI via software.
Também é fornecido um template para nodos sensores, o exemplo [CANSensor.ino](https://github.com/KakiArduino/MCP2515/tree/version_1/example/CANSensor).


*******
Sumário:
 1. [Variáveis de um frame](#frames_var)   
 2. [Declaração de frames](#frames_fun)
 3. [Variaives públicas](#MCP_var)
 4. [Funções públicas](#MCP_fun)

*******

<div id='frames_var'/> 

## Variáveis de um frame

### Tipo de frames 
* `String type = "Unknown";`<br/>
String com o tipo do frame, padrão ("Std. Data"), estendido ("Ext. Data") ou "No frame" que é usado para indicar que há novos frames nos buffers de entrada do MCP2515.

### Bytes

<div id='frames_var_bts'/> 

* `uint8_t bts[14];`<br/>
Lista com todos (no máximo 14) os bytes do frame, no formato apresentado na figura [Frame](#frame), 2 bytes para o ID padrão, 3 bytes para a extensão de ID, 1 byte para o código de comprimento de dado (DLC - data len code) e de 0 a 8 bytes para o dado. O comprimento da lista pode ser alterar em função do tamanho do dado.

### ID padrão
* `uint16_t id_std;`<br/>
Variável de 2 bytes que armazena o ID padrão do frame. Também pode-se construir o valor do ID padrão apartir dos dois primeiros bytes da lista [bts](#frames_var_bts), o `bts[0]` e `bts[1]`.

### Extensão de ID
* `uint32_t id_ext;`<br/>
 Variável de 4 bytes que armazena a extensão de ID do frame. Também pode-se construir o valor da extensão de ID apartir dos bytes `bts[2]`, `bts[3]` e `bts[4]` da lista [bts](#frames_var_bts).

### Código de comprimento de dados - DLC
* `uint8_t dlc;`<br/>
Variável de 1 byte que armazena o código de comprimento do dado (DLC - data len code), número de bytes de dados no frame. O DLC pode ser encontrado também no `bts[5]` da lista [bts](#frames_var_bts).

### Dados
* `uint8_t data[8];`<br/>
Lista com os bytes (de 1 a 8) de dados do frame. Os bytes de dados também pode ser acessados das possições `bts[6]` até `bts[13]` da lista [bts](#frames_var_bts).

 
<div id='frames_fun'/>

## Declaração de frames

### Frames genéricos
* `CANframe();`<br/>
Função para declaração de uma estrutura de variáveis do tipo CANframe sem argumentos.

Exemplo de uso:
```C++
CANframe frm();
frm.id_std = 0x7FF;
```
>  Declaração de um frame sem fornecer parâmetros de entrada, seguida, na linha de baixo, pela atribuição de 0x7FF para o ID padrão, o maior valor possível, do frame criado acima.


* `CANframe(uint8_t *frameBytes, uint8_t extFlag = 0);`<br/>
Função para criação de frame, a partir de uma lista com todos os bytes do frame.

Parâmentros de entrada:
  1. **frameBytes**: lista com todos os bytes do a serem atribuídos ao frame.
  2. **extFlag**: sinalizador de extensão de ID, se 0 o frame é padrão, se 1 é estendido.

Exemplo de uso:
```C++
frameBytes [10] = {0x07, 0xFF, 
                   0x03, 0xFF, 0xFF,
                   0x04,
                   0x3, 0xFF, 0xF7, 0x21};
            
CANframe frm(frameBytes, 1);
Serial.println(frm.id_std);
```
>  Primeiro é declarado a lista *frameBytes*, os dois primeiros bytes (0 e 1) compõem o ID padrão, concatenando tem-se 0x7FF o maior valor possível. 
>  Os três bytes seguintes da lista *frameBytes*, posições 2, 3 e 4, correspondem a extensão de ID, que neste caso vale o valor máximo 0x3FFFF.
>  O byte seguinte, posição 5, é o código dlc, que no caso indica quatro bytes de dados.
>  Após a declaração da lista com os bytes, é feito a declaração do frame estendido com a lista *frameBytes*.
>  Por fim o ID padrão é impresso através da variável *id_std* na última linha, isto é possível, pois ao declarar o frame, todos suas variáveis são atribuídas.

### Frames Estendidos

* `CANframe(uint16_t idstd, uint32_t idext, uint8_t dlc_, uint8_t *data);`<br/>
   Função para criação de frame estendido com declaração do ID padrão, extensão de ID, DLC - Data Len Code e uma lista de bytes de dados, **data**.

<div id='CANframe_ext'/>  

Parâmetros de entrada:
1.**idstd**: variável de 2 bytes onde deve ser informado o valor do ID padrão do frame, no máximo 0x7FF.
2.**idext**: variável de 4 bytes onde deve ser informado o valor da extensão de ID do frame, no máximo 0x3FFFF.
3.**dlc_**: é o número de bytes de dados.
4.**data**: é a lista que contem os bytes de dados dos frames.

Exemplo de uso:
```C++
uint8_t data[2] = {0, 10};
CANframe frm(1, 6, 2, data);
```
> Na primeira linha é declarada uma lista com os bytes de dados.
> Seguido, na linha abaixo, da declaração de um frame estendido com ID padrão igual a 1, extensão de ID igual a 6, e com dois bytes de dados (dlc_ = 2), sendo 0 e 10.

* `reload(uint16_t idstd, uint32_t idext, uint8_t dlc_, uint8_t *data);`<br/>
Função para recarregar um frame qualquer com estendido, atribuindo os valores fornecidos com entrada nas variáveis correspondentes.
Os parâmetros de entrada são os mesmos, e em mesma ordem, da função [CANframe(idstd, idext, dlc_, data)](#CANframe_ext), descrita um pouco acima.

Exemplo de uso:
```C++
uint8_t data[2] = {0, 10};
CANframe frm(1, 2, data);
frm.reload(1, 10, 2, data);
```
> As duas primeiras linhas deste exemplo já foram apresentadas como exemplo para a função [CANframe(idstd, dlc_, data)](#CANframe_pad), descrito anteriormente, nele é criado um frame padrão.
> Na última linha o frame frm é recarregado como estendido com a função *reload(..)*, a única diferença é a inserção do valor da extensão de ID (10).


### Frames Padrão

* `CANframe(uint16_t idstd, uint8_t dlc_, uint8_t *data);`<br/>
Função para criação de frames padrões, devem ser informados o ID padrão, o DLC, e de uma lista com os bytes dos dados.

<div id='CANframe_pad'/> 

Parâmetros de entrada:
1. **idstd**: variável de 2 bytes onde deve ser informado o valor do ID padrão do frame, no máximo 0x7FF.
2. **dlc_**: número de bytes de dados.
3. **data**:  lista contendo os bytes de dados dos frames.

Exemplo de uso:
```C++
uint8_t data[2] = {0, 10};
CANframe frm(1, 2, data);
```
> Na primeira linha é declarada uma lista com os bytes de dados.
> Seguido, na linha abaixo, da declaração de um frame padrão com ID padrão igual a 1, e com dois bytes de dados (dlc_ = 2), sendo 0 e 10.

* `reload(uint16_t idstd, uint8_t dlc_, uint8_t *data);`<br/>
Função para recarregar um frame qualquer com padrão, atribuindo os valores fornecidos com entrada nas variáveis correspondentes.
Os parâmetros de entrada são os mesmos, e em mesma ordem, da função [CANframe(idstd, dlc_, data)](#CANframe_pad), descrita um pouco acima.

Exemplo de uso:
```C++
uint8_t data[2] = {0, 10};
CANframe frm(1, 10, 2, data);
frm.reload(1, 2, data);
```
> Na primeira linha é declarada uma lista com dois bytes de dados.
> Na segunda linha um frame estendido é declarado, tendo o ID padrão 1, a extensão de ID 10, o DLC 2, e os dados atribuídos a lista data na linha de cima.
> Na última linha o frame *frm* é recarregado com padrão, com a função *reload(..)*, a diferença é a ausência da extensão de ID.


### Somente os dados

* `reload(uint8_t dlc_, uint8_t *data_);`<br/>
Função para recarregar o campo de dados de frame qualquer, essa função não altera os valores de ID, também pouco o tipo de frame, também é alterado o DLC.
Essa função pode ser usada em um loop, onde os dados do frame são atualizados periodicamente, mas seus valores de ID não.

Parâmetros de entrada:
1. **dlc_**: número de bytes de dados.
2. **data_**:  lista contendo os bytes de dados dos frames.

Exemplo de uso:
```C++
uint8_t data[4] = {0, 10, 10, 0};
frm.reload(4, data);
```
> Na primeira linha é declarada uma lista com quatro bytes de dados.
> Na segunda linha um frame anteriormente criado (frm) é recarregado com a lista de bytes de dados data.

<div id='MCP_var'/>  

## Variaives públicas

<div id='MCP_var_SPI'/>  

### SPI

* `unsigned long int SPI_speed = 10000000;`<br/>
Máxima frequência do clock do SPI usado na comunicação entre o Arduino e o MCP2515, que é limitada pelo último em 10 M Hz. O valor praticado pelo Arduino é definido em função do seu clock interno e do limite fornecido em *SPI_speed*.
    
* `uint8_t SPI_wMode = 0;`<br/>
Modo de operação da comunicação SPI entre o Arduino e o MCP2515, o controlador CAN suporta dois modos o [0,0] (0) e o [1,1] (3).
    
 * `uint8_t SPI_cs;`<br/>
Número do pino do Arduino usado como *chip select* na comunicação SPI entre o Arduino e o MCP2515.

### Configurações gerais do MCP2515

<div id='MCP_var_conf'/>  

 * `uint8_t crystalCLK = 8;`<br/>
Frequência do oscilador que fornece a base de clock para o MCP2515, em mega hertz (M Hz), valor padrão é 8 M Hz. Verifique os valores [pré-definidos](#crystal_pre).
    
 * `uint16_t bitF = 125;`<br/>
Taxa de bits do barramento, em k bit/s, valor padrão é 125 k bit/s. Verifique os valores [pré-definidos](#crystal_pre).
    
 * `uint8_t wMode = 0x00;`<br/>
Está variável é usada para manipular o registro 0x0F do controlador CAN, 0x00 é modo de operação padrão da biblioteca, modo de operação *Normal*, encerra a solicitação para cancelar o envio de todas as transmissões, modo *One-Shot* desabilitado, pino *CLKOUT* do MCP25 desabilitado e seta o *CLKOUT = System Clock/1*.
Para outras opções consultar o [datasheet](http://ww1.microchip.com/downloads/en/DeviceDoc/MCP2515-Stand-Alone-CAN-Controller-with-SPI-20001801J.pdf).
    
* `uint8_t RXB0CTRL = 0x66;`<br/>
Está variável é usada para manipular o registro 0x60 do MCP2515, ele configura o buffer de entrada *RXB0*.
O valor padrão 0x66, desabilita as mascaras e filtros e ativa o rollover do buffer de entrada *RXB0* para o *RXB1*.
    
* `uint8_t RXB1CTRL = 0x60;`<br/>
Está variável é usada para manipular o registro de configuração do buffer de saída *RXB1* do MCP2515 (0x70).
O valor padrão 0x60, desabilita as mascaras e filtros.
        
    
* `uint8_t TXB0CTRL = 0x00;`<br/>
Está variável manipula o registro 0x30, que corresponde a configuração do buffer de saída *TXB0* do MCP2515.
O valor padrão 0x00, atribui o nível mais baixo de prioridade (0) e aborta possível frame alocado buffer *TXB0*.
 > Obs.: Quando todos os buffers de saída possuem a mesma prioridade o, o TXB0 se torna o menos prioritário.
    
* `uint8_t TXB1CTRL = 0x00;`<br/>
Está variável manipula o registro 0x40, que corresponde a configuração do buffer de saída *TXB1* do MCP2515.
O valor padrão 0x00, atribui o nível mais baixo de prioridade (0) e aborta possível frame alocado buffer *TXB0*.
> Obs.: Quando todos os buffers de saída possuem a mesma prioridade, o *TXB1* se torna mais prioritário do que o *TXB0* e menos prioritário do que o *TXB2*.
        
* `uint8_t TXB2CTRL = 0x00;`<br/>
Está variável manipula o registro 0x50, que corresponde a configuração do buffer de saída *TXB2* do MCP2515.
O valor padrão 0x00, atribui o nível mais baixo de prioridade (0) e aborta possível frame alocado buffer *TXB0*.
> Obs.: Quando todos os buffers de saída possuem a mesma prioridade, o *TXB2* se torna o mais prioritário.
        
* `uint8_t CANINTE = 0xBF;`<br/>
Está variável manipula o registro 0x2B do MCP2515.
O valor padrão 0xBF no registro 0x2B habilita interrupções, quando uma mensagem é recebida no *RXB0* ou no *RXB1*, quando qualquer um dos buffers de saída  (*TXB0*, *TXB1* e *TXB2*) ficar vazio, quando ocorrer erros de múltiplas fontes indicadas no registro *EFLG* do MCP2515, e quando uma transmissão de mensagem for interrompida.
    
* `uint8_t BFPCTRL = 0x0F;`<br/>
Está variável manipula o registro 0x0C do MCP2515.
O valor padrão 0x0F no registro 0x0C configura os pinos do MCP2515, o *RX0BF* é habilitado e configurado como interrupção que indica quando uma nova mensagem valida é carregado no *RXB0*, e de forma análoga o *RX1BF* é configurado para o buffer *RXB1*. 
    
* `uint8_t TXRTSCTRL = 0x00;`<br/>
Está variável manipula o registro 0x0D do MCP2515.
O valor padrão 0x00 não habilita nenhum interrupção com relação os pinos TX0RTS, TX1RTS e TX2RTS.        
        
* `uint8_t CNF1 = 0x00;`<br/>
Está variável salva o valor grava no registro 0x2A do MCP2515, ele faz parte da configuração do Bit-Timing.
Há algumas opções pré definidas, estas podem ser configuradas atribuindo valores as variáveis *crystalCLK* e *bitF*, e dessa forma as variáveis *CNF1*, *CNF2* e *CNF3* são atualizadas durante a inicialização feita pela função *begin()* ou pela função de configuração específica, *confCAN()*.

<div id='crystal_pre'/>  

Os casos pré-definidos são: 
1. *crystalCLK* = 4: com duas possíveis taxas 125 k bit/s e 250 k bit/s.

2. *crystalCLK* = 8: com três possíveis taxas 125 k bit/s, 250 k bit/s e 500 k bit/s.
> As distancias estimas foram calculadas (não testadas) para o cristal de 8 M Hz, como sendo de até 275 m para 125 k bit/s, 125 m para 250 k bit/s e 50 m para 500 k bit/s.

3. *crystalCLK* = 20: com quatro possíveis taxas 125 k bit/s, 250 k bit/s, 500 k bit/s e 1000 k bit/s.

Para outros casos deve-se alterar *CNF1* diretamente, mais detalhes consultar o [datasheet](http://ww1.microchip.com/downloads/en/DeviceDoc/MCP2515-Stand-Alone-CAN-Controller-with-SPI-20001801J.pdf).
    
* `uint8_t CNF2 = 0x42;`<br/>
Está variável salva o valor gravado no registro 0x29 do MCP2515. 
Sua configuração segue de forma análoga a da varaiável *CNF1*.

* `uint8_t CNF3 = 0x02;`<br/>
        Está variável salva o valor grava no registro 0x28 do MCP2515.
Sua configuração segue de forma análoga a da varaiável *CNF1*.


### Filtros e Mascaras

<div id='MCP_var_filMask'/>  

* `uint16_t MASstd[2] = {0x00, 0x00};`<br/>
Lista com os valores dos ID padrões das mascaras 0 (*MASstd[0]*) e 1 (*MASstd[1]*) do MCP2515.
    
* `uint32_t MASext[2] = {0x00, 0x00};`<br/>
Lista com os valores das extensões de ID das mascaras 0 (*MASext[0]*) e 1 (*MASext[1]*) do MCP2515.
    
* `uint16_t FILstd[6] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x00};`<br/>
Lista com os valores de ID padrão dos filtros do MCP2515, *FILstd[0]* corresponde ao filtro 0, e assim por diate, até *FILstd[6]* que correnponde ao filtro 6.
    
* `uint32_t FILext[6] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x00};`<br/>
Lista com os valores das extensões de ID dos filtros do MCP2515, *FILext[0]* corresponde ao filtro 0, e assim por diate, até *FILext[6]* que correnponde ao filtro 6.


### Erros

<div id='MCP_var_erros'/>  

* `String errLog = "no error";`<br/>
Armazena o último erro genérico ocorrido. Pode-se atualizar o valor de *errLog* chamando a função *errCont()*.
    
* `String errMode = "Error Active";`<br/>
Armazena o modo de confinamento de erro,  "Error Active", "Error Passive" ou "Bus-Off". Pode-se atualizar o valor de *errMode* chamando a função *errCont()*.
    
* `uint16_t RX0OVR = 0;`<br/>
Armazena o número de *overload* no buffer *RXB0*. Pode-se atualizar o valor de *RX0OVR8 chamando a função *errCont()*.

* `uint16_t RX1OVR = 0;`<br/>
Armazena o número de overloads no buffer *RXB1*. Pode-se atualizar o valor de *RX1OVR* chamando a função *errCont()*.
        
* `uint8_t multInt = 0;`<br/>
Armazena o número de erros de multiplicas fontes detectados, indicados pelo registro *ERROR FLAG REGISTER* (0x2D).Pode-se atualizar o valor de *multInt* chamando a função *errCont()*.
    
* `uint8_t  MERRF = 0;`<br/>
Armazena o número de erros de mensagens detectados. Pode-se atualizar o valor de *MERRF* chamando a função *errCont()*.
        
* `uint8_t  WAKIE;`<br/>
Armazena o valor da *Wake-up flag* de MCP2515. Pode-se atualizar o valor de *WAKIE* chamando a função *errCont()*.
    
* `uint8_t TEC = 0;`<br/>
Armazena o valor do contador de erros de transmissão TEC. Pode-se atualizar o valor de *TEC* chamando a função *errCont()*.
    
* `uint8_t REC = 0;`<br/>
Armazena o valor do contador de erros de transmissão REC. Pode-se atualizar o valor de *REC* chamando a função *errCont()*.


### Frames

<div id='MCP_var_frm'/>  

* `CANframe frameRXB0;`<br/>
Variável do tipo *CANframe*, criada para receber os frames recebidos no buffer *RXB0*. Pode-se atualizar o valor de *frameRXB0* chamando a função *readFrame()*.
        
* `CANframe frameRXB1;`<br/>
Variável do tipo *CANframe*, criada para receber os frames recebidos no buffer *RXB1*. Pode-se atualizar o valor de *frameRXB1* chamando a função *readFrame()*.

## Funções públicas

<div id='MCP_fun'/>  

### Declaração

<div id='MCP_fun_mcp'/>  

* `mcp.MCP(uint8_t spi_cs, unsigned long int spi_speed = 10000000, uint8_t spi_wMode = 0);`<br/>

A função *MCP2515(...)* criar um objeto MCP2515, que herda as funções da biblioteca MCP2515.
Todos os parâmetros de entrada da função *MCP2515(...)* são relacionados a comunicação SPI, e são listados abaixo, juntamente de dois exemplos de uso.

Parâmetros de entrada:
1. **spi_cs**:  é o número do pino do Arduino usado como *chip select* do MCP2515.
2. **spi_speed**:  é a máxima frequência do clock da comunicação SPI, seu valor padrão é 10000000.
>  Obs.: Esse é o valor máximo suportado pelo MCP2515 e serve apenas como limite superior, a frequência do clock é setada automaticamente pelo Arduino, dentro do limite informado.
3. **spi_wMode**: indica o modo de operação do SPI, o MCP2515 suporta o modo [0,0] e o modo [1, 1], que equivalem respectivamente, o modo 0 e  ao modo 3 do Arduino ([SPI - Arduino](https://www.arduino.cc/en/reference/SPI)). O valor padrão é 0.

Exemplos de uso:

Declaração de um objeto MCP2515.
```C++
#include <MCP2515_1.h>
MCP2515 mon(4);
```
> Neste exemplo foi, na primeira linha, incluído a versão 1 da biblioteca MCP2515, através do arquivo [MCP2515_1](https://github.com/KakiArduino/MCP2515). 
> Na segunda linha foi declarado um objeto *MCP2515* chamado *mon*, que tem como *chip select* o pino digital 4 do Arduino.

Outro exemplo de declaração de objeto MCP2515.
```C++
MCP2515 mcp(7, 10000000, 3);
```
> Desta vez foi declarado um objeto chamado **mcp**, que tem como *chip select* o pino digital 7 do Arduino, e usa o modo 3 do SPI do Arduino, que equivale [1, 1].



### Inicialização e configuração

<div id='MCP_fun_mcp'/>  

* `mcp.begin();`<br/>
A função *begin()* inicializa a comunicação SPI entre o Arduino e o MCP2515, e configura o controlador CAN para operar de acordo com os valores setados nas variáveis de configuração.
As variáveis de configuração, bem como seus valores padrões, foram descritas na secção anterior em [Configurações gerais do MCP2515](#MCP_var_conf) .

Exemplos de uso:

Parte inicial da função *void setup()* do exemplo [CANMon.ino](https://github.com/KakiArduino/MCP2515/blob/version_1/example/CANMon/CANMon.ino).
```C++
void setup() {
  Serial.begin(9600);
  mon.bitF = 125; // k bits/s
  mon.begin();
```
> Na penúltima linha é setado a taxa de bits para 125 k bit/s, *bitF* é a variável de controle que armazena o valor da taxa bits. 
> Na última o MCP2515 é inicializado e configurado com a função *mon.begin()*.

Configuração padrão.
```C++
sensor.begin();
```
> Neste exemplo o MCP2515 é inicializado e configurado no modo padrão, *sensor* é o nome do objeto.


### Reset 

<div id='MCP_fun_mcp'/>  


* `mcp.reset();`<br/>
A função *reset()* não possui argumentos, ela reinicia o CI MCP2515 enviando a instrução 0xC0 pela comunicação SPI.
Atenção ao voltar do reset o CI MCP2515 se encontra em modo de configuração e com valores padrão nos registros, e deve-se esperar algo entorno de 2 micro secondos antes de usa-lo ([datasheet](http://ww1.microchip.com/downloads/en/DeviceDoc/MCP2515-Stand-Alone-CAN-Controller-with-SPI-20001801J.pdf)), isso pode ser feito através da função [**delayMicroseconds()**](https://www.arduino.cc/reference/en/language/functions/time/delaymicroseconds/) do Arduino.
É aconselhável o uso dessa função logo após a inicialização do CI, e antes de configurá-lo, pois assim tem-se certeza dos valores salvos em seus registros.

### Read

<div id='MCP_fun_read'/>  

* `mcp.read(uint8_t REG, uint8_t *data, uint8_t n = 1);`<br/>
A função *read(...)* realiza *n* (de 1 à 128) leituras sequencias de registros do MCP2515, a partir do registro *REG* informado, e aloca os *n* bytes em uma lista previamente criada e indicada pelo endereço *data*,  a lista *data* deve ter o tamanho de *n* *bytes*.
Abaixo segue a descrição dos parâmetros de entradas da função *read(...)*, e na sequencia dois exemplos, um lendo um registro e o outro lendo *13* registros.


Parâmetros de entrada:
1. **REG**: é o endereço do primeiro registro do MCP2515 a ser lido, deve ter um tamanho de 1 byte e seu valor vai 0x0 (0) até 0x80 (128).
2. **data**: é o endereço da lista criada para armazenamento dos valores lidos. A lista *data* deve ter o tamanho de *n* bytes.
3. **n**: é o número de registros a serem lidos, contando a partir do *REG*, por padrão *n =1*, logo se não alterado a função *read(..)* lerá apenas um registro.

Exemplos de uso:

Leitura do registro TEC do MCP2515.
```C++
uint8_t data[1];
mcp.read(0x1C, data);
```
> Neste exemplo foi primeiro declaro uma lista (*data[1]*) com 1 byte e na sequencia é realizado a leitura do registro *0x1C*, que armazena a contagem de erro de transmissão (TEC) do MCP2515.

Leitura do buffer de entrada *RXB1* do MCP2515.
```C++
uint8_t data[13];
mcp.read(0x61, data, 2);
```
> Neste exemplo foi primeiro declaro uma lista com 13 bytes, e na sequencia é realizado a leitura dos 13 registros do buffer de entrada *RB1* do MCP2515.


### Verificação de valor em registros

<div id='MCP_fun_regCheck'/>  

* `mcp.regCheck(uint8_t REG, uint8_t VAL, uint8_t extraMask = 0xFF);`<br/>

A função *regCheck(...)* confere se o byte salvo no registro *REG* é igual ao byte *VAL*, se for a função retorna 0, se não ela retorna 1. Se não for possível escrever no registro *REG* a função retorna 2.
Pode-se usar essa função também para verificar o valor de um ou mais bits do byte armazenado em *REG*, para isso deve-se informar a mascara (*extraMask*) capaz de selecionar os bits desejados pela operação lógica *&* (and).
Essa função pode ser chamada em conjunto com a função *write(...)*, de modo que seja feita uma verificação do sucesso da escrita no registro.
A seguir a descrição dos parâmetros de entrada da função *regCheck(...)* e dois exemplos de uso, sem e com mascara de seleção de bits.

Parâmetros de entrada:
1. **REG**:  é o endereço do registro do MCP2515 a ser verificado. Deve ter um tamanho de 1 byte e seu valor vai 0x0 (0) até 0x80 (128).
2. **VAL**:  é o byte que se deseja verificar em *REG*.
3. **extraMask**:  é uma mascara de 1 byte, para selecionar os bits desejados pela operação logica *&* (and). Por padrão *extraMask = 0xFF*, logo por padrão a verificação é feita sobre todos os bits.

Exemplos de uso:

Verificação do registro TEC do MCP2515.
```C++
uint8_t check = 0;
check = mcp.regCheck(0x1C, 0);
if(check =! 0){
  Serial.println("TEC > 0");
}
```
> Neste exemplo primeiro é declaro a variável check para armazenar o retorno da função *regCheck(...)*.
> Depois é chamada a função *regCheck(0x1C, 0)*, que verificará se o valor salvo no registro *0x1C* é 0.
> O registro *0x1C* armazena a contagem dos erros de transmissão (TEC) do MCP2515.
> O valor retornado pela função *regCheck(...)* e salvo na variável check é comparado com 0, se ele for diferente de zero, significa que TEC possui um valor maior que zero, e ao menos um erro de transmissão foi detectado.

Verificando se o MCP2515 está em modo de configuração.
```C++
uint8_t check = 0;
check = mcp.regCheck(0x0F, 0x80, 0xE0);
if(check == 0){
    //realizar as configuração desejadas aqui.
}
```
> Dessa vez a função *regCheck(...)* é usado com mascara, para verificar se o MCP2515 está no modo de configuração.
> O código do modo de operação é armazenado nos três bits mais significantes do registro 0x0F, por isso o uso da mascara 0xE0. 
> Se o MCP2515 estiver no modo de configuração o valor do registro 0x0F, após aplicação da mascara 0xE0, deve ser 0x80.


### Escrita em regisitros

<div id='MCP_fun_write'/>  

* `mcp.write(uint8_t REG, uint8_t VAL, uint8_t CHECK = 1);`<br/>
A função *write(...)* escreve o byte *VAL* no registro *REG* e pode verifica se o valor foi realmente escrito. 
O sucesso pode ser verificado na variável *errLog*, no caso de falha ela valerá "Erro na escrita", se seu valor for "Reg invalido", o registro *REG* não pode ser escrito. 
A checagem da escrita é opcional, e por padrão é feita, para que ela não seja feita basta fornecer 0 para *check*. Desabilitar a checagem fará com que a rotina *write(...)* execute em menos tempo, visto que não será feita a checagem.
A seguir a descrição dos parâmetros de entrada da função *write(...)*, e dois exemplos de utilização, com e sem verificação de escrita.
> Obs.: Há alguns registros que só podem ser escritos se o MCP2515 estiver no modo de configuração.


Parâmetros de entrada:
1. **REG**: é o endereço do registro do MCP2515 para escrita. Deve ter um tamanho de 1 byte e seu valor vai 0x0 (0) até 0x80 (128).
2. **VAL**: é o byte que se deseja escrever em *REG*. Deve ter o tamanho de 1 byte.
3. **check**: é a sinalização de checagem, se fornecido 1 para *check* a função *write(...)* realizará a checagem, se fornecido 0 a função *write(...)* não realizará a checagem. Por padrão a checagem é feita.

Exemplo de uso:

Escrevendo no registro 0x36 com verificação automática.

```C++
mcp.write(0x36, 0xFE);
if(errLog=="Erro na escrita"){
    Serial.println(mcp.errLog)
}
}
```
> Neste exemplo o byte 0xFE é escrito no registro 0x36.
> Por padrão a função *write(...)* verifica o sucesso da escrita, o que é feito comparando o valor da variável *errLog*.
> O registro 0x36 pertence ao buffer de saída *TXB1* do MCP2515.

Escrevendo no registro 0x36 sem verificação.

```C++
mcp.write(0x36, 0xFE, 0);
```
> Desta vez a checagem é desabilitada informando o valor 0 para o parâmetro *check*.



### Alteração de bits em registros

<div id='MCP_fun_bitModify'/>  

* `mcp.bitModify(uint8_t REG, uint8_t MASK, uint8_t VAL, uint8_t check = 0);`<br/>

A função *bitModify(...)* é usada para alterar valores de bits específicos no registro *REG* do MCP2515.
Para tal deve-se fornecer um mascara (MASK, 1 byte) capaz de selecionar os bits que se deseja modificar pela operação lógica *&* (and). 

Parâmetros de entrada:

1. **REG**: é o endereço do registro do MCP2515, onde se quer alterar valores de bits. Deve ter um tamanho de 1 byte e seu valor vai 0x0 (0) até 0x80 (128).
2. **MASK**: é a mascara para selecionar a posições de bits que deseja-se modificar. Os bits são selecionados pela operação lógica é a *&* (and). Deve ter 1 byte de tamanho.
3. **VAL**:  é o byte que contem o valor dos bits a serem escritos no registro *REG*. Deve ter 1 byte de tamanho.
4. **check**:  é a sinalização de checagem, para está função o padrão é não checar e por isso *check = 0*. Caso queira verificar o sucesso da alteração dos bits fornece 1 para check, a habilitação da checagem prolonga o tempo de execução.

Exemplos de uso:

Setar o modo de operação *Listen-Only*
```C++
mcp.bitModify(0x0F, 0xE0, 0x60);
```
> Neste exemplo o MCP2515 é setado no modo *Listen-Only*, alterando o valor dos três bits mais significantes do registro 0x0F, com a mascara 0xE0, para 0 1 1.

Setar o modo de operação \textit{Sleep}
```C++
mcp.bitModify(0x0F, 0xE0, 0x20, 1);
```
> Desta vez o MCP2515 é setado para o modo *Sleep*, alterando os três bits mais significantes do registro 0x0F para 0 1 0. Além disso é feita a verificação fornecendo 1 para o parâmetro *check*.


### Configuração do modo de operação

<div id='MCP_fun_conf'/>  

* `mcp.confMode();`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `mcp.confRX();`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```

* `mcp.confTX();`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `mcp.confINT();`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```



* `mcp.confFM();`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `mcp.confCAN();`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `mcp.status(uint8_t *status);`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `mcp.errCont();`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `writeID(uint16_t sid, uint8_t id_ef = 0, uint32_t eid = 0, uint8_t txb = 0, uint8_t timeOut = 10, uint8_t check = 0);`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `mcp.loadTX(uint8_t *data, uint8_t n = 8,  uint8_t abc = 1);`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `mcp.send(uint8_t);`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```


* `writeFrame(CANframe frameToSend, uint8_t txb_ = 0, uint8_t timeOut = 10, uint8_t check = 0)`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```

* `mcp.abort(uint8_t abortCode = 7);`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```

* `mcp.readID(uint8_t *id, uint8_t rxb = 0);`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```

* `mcp.readFrame()`<br/>

Parâmetros de entrada:
1. ** **:
2. ** **:
3. ** **:

Exemplo de uso:
```C++

```



* `mcp.digaOi(char *oi);`<br/>

Parâmetros de entrada:
1. **oi**: é o char que informa quais variáveis serão impressas, se informado "status", será impresso somente as variáveis relacionadas ao estado do MCP2515 e seus buffers de saída e entrada. Se informado "eros" ou "errors", serão impressos valores de erros. Se informado "SPI", serão impressos os parâmetros do SPI. Se informado "CAN", serão impressos os parâmetros da configuração da CAN. Se informado "controle", serão impressos os parâmetros de controle do MCP2515. Se informado "filtros", serão impressos os filtros setados no MCP2515. Se informado "filtros", serão impressos as mascaras setados no MCP2515.

Exemplos de usos:

```C++
Serial.begin(9600);
mcp.digaOi();
```
> Neste exemplo a função *digaOI()* imprimirá todas as variáveis.

```C++
Serial.begin(9600);
mcp.digaOi("error");
```
> Exemplo de uso da função *digaOi("error")* imprimindo somente os parâmetros relacionados a erros.
