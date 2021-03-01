# COMO FUNCIONA O PWM NO 

## O que tem que ser manipulado para escrita de pinos analógicos

Para a escrita em pinos analógicos devemos retornar para o pino um valor de tensão entre 0 e 5V. Esse processo é feito usando a variação dos pulsos 
digitais para que o valor de tensão médio seja o desejado na saída do pino. Essa modulação na largura do pulso é conhecida como PWM que vem do inglês Pulse Width Variation.

### Registrador TCCR0A/TCCR0B

Esses são os registradores de controle do contador/timer A e B de 8bits de resolução.

![image](https://user-images.githubusercontent.com/32770973/109569966-0f14c780-7ac8-11eb-82cb-13900b38ec98.png)

![image](https://user-images.githubusercontent.com/32770973/109571646-bb57ad80-7aca-11eb-91f8-48596115b7eb.png)

para o nosso projeto vamos escolher o modo fast PWM, não invertido necessário para a escrita na porta digital a saída do canal verde do LED RGB. Dessa forma
nós vamos setar os seguintes bits.

![image](https://user-images.githubusercontent.com/32770973/109572052-55b7f100-7acb-11eb-9b3b-2acc37cd9af7.png)

primeiramente vamo setar os bits do seletor CS02:00 de acordo com a tabela a seguir:

![image](https://user-images.githubusercontent.com/32770973/109572392-e1318200-7acb-11eb-976d-69c15b88579c.png)

diferentemente do timer que setamos anteriormente não há problema de setar esses seletores primeiro, pq o pulso não precisa ser contado, apenas o tamanho da largura 
do pulso que temos que analisar para que a saída apresente a tensão que querermos.

---

Ainda nesses registradores vamos configurar o modo de geração do formato de onda. Para o nosso caso como já citado acima escolhemos o modo fast PWM, não invertido e para isso
precisamos setar os bits WGM02:00:

![image](https://user-images.githubusercontent.com/32770973/109572835-af6ceb00-7acc-11eb-8056-032fcce7d5d6.png)

de acordo com a tabela:

![image](https://user-images.githubusercontent.com/32770973/109573036-0672c000-7acd-11eb-8c6f-1d8e32e3308f.png)

e também setar os bits COM0A01:00 de acordo com a tabela abaixo

![image](https://user-images.githubusercontent.com/32770973/109574326-7897d480-7ace-11eb-98a3-c5269e796fee.png)









### Exemplo de código em alto nível de conversão de um canal AD

```c
funcao principal(canal){   //número de 4 bits
  REGS recebe "01";        //para referencia de AVcc com um capacitor externo em AREF
  ADLAR recebe '1';        //para mover os dados pós conversao a esqueda
  ADPS recebe "111";       //para selecionar o divisor de clock por 128
  MUX recebe canal;        //seleção de canal de entrada de dados
  ADEN recebe '1';         //enable do ADC setado
  ADSC recebe '1';         //iniciar a conversão
  enquanto (ADIF igual 0); //espera o bit de flag ser acionado
  ADIF recebe 0;           //reseta a flag
  ADEN recebe '0';         //enable do ADC resetado
}
```

Programa para fazer uma conversão AD.

## REFERÊNCIAS

Electrosaurio. **ADC SIN ARDUINO ATMEGA328P - CONVERSOR ANALÓGICO A DIGITAL**. Youtube, 27 mar. de 2019. Disponível em <https://www.youtube.com/watch?v=SwjVAwsi8Os&ab_channel=Electrosaurio>. Acesso em: 26 de fevereiro de 2021.
Atmel. **8-bit AVR Microcontroller with 32K Bytes In-System Programmable Flash - DATASHEET.**. Disponível em <https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf>. Acesso em: 23 de fevereiro de 2021.
