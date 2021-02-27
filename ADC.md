# COMO FUNCIONA O MÓDULO ADC DO ATMEGA328P

## O que tem que ser manipulado para leitura e escrita de pinos analógicos

### Registrador ADCSRA

O módulo ADC tem uma frequencia de operação entre 50kHz a 200kHz o que indica que temos que fazer uma divisão de clock do clock geral do ATMega328P.

![image](https://user-images.githubusercontent.com/32770973/109388344-365f6f00-78e5-11eb-88c4-26f82ca2f3bc.png)

Essa divisão de clock é feita no registrador ADCSRA, mais expecificamente nos bits ADPS2, ADPS1, ADPS0 (seletores que indicam qual vai ser a divisão de clock de acordo com a tabela abaixo).

![image](https://user-images.githubusercontent.com/32770973/109387796-2c883c80-78e2-11eb-8304-830105cb960d.png)

---

Ainda falando sobre o registrador ADCSRA temos outro bit muito importante que é o ADEN, o enable do conversor AD.

![image](https://user-images.githubusercontent.com/32770973/109388356-568f2e00-78e5-11eb-8b5b-da819dae556e.png)

(0 desligado, 1 ligado).

---

Outro bit importante do registrador ADCSRA é o ADSC que é responsável pela conversão do valor de tensão de entrada em um número de 10 bits.

![image](https://user-images.githubusercontent.com/32770973/109388513-59d6e980-78e6-11eb-89ec-6be1e11736a8.png)

(0 desligado, 1 ligado) Se esse bit é setado ao mesmo tempo do bit de enable (ADEN) a conversão é feita em 25 ciclos de clock.

---

Por fim podemos falar também do bit ADIF, que é o bit de flag que sinaliza quando a conversão AD foi finalizada.

![image](https://user-images.githubusercontent.com/32770973/109388677-57c15a80-78e7-11eb-9ba0-a5e52faba71e.png)

Ressaltando que esse bit de flag tem que ser monitorado após a instrução de início da conversão para poder executar outras instruções somente após a conversão ter sido concluída.

---

### Registrador ADMUX

Outro registrador a ser manipulado é o ADMUX, que podemos começar falando sobre os bits REFS1 e REFS0.

![image](https://user-images.githubusercontent.com/32770973/109389177-2a29e080-78ea-11eb-9965-f13969778d8f.png)

esses bits indicam qual será a referência usada pelo conversor AD para fazer as suas conversões (possiveis referências encontram-se na tabela abaixo).

![image](https://user-images.githubusercontent.com/32770973/109389065-788aaf80-78e9-11eb-8e3c-8a172225aff5.png)

---

Um bit que pode ser usado, mas não é de suma importância é o ADLAR.

![image](https://user-images.githubusercontent.com/32770973/109389199-40d03780-78ea-11eb-9002-7257f1728b22.png)

Esse bit simboliza o deslocamento ou não dos valores dos bits ADC pós conversão. Caso setado desloca a esquedas os bits da conversão nos REGISTRADORES ADCH e ADCL (configurações dos pinos podem ser vistas nas tabelas abaixo).

![image](https://user-images.githubusercontent.com/32770973/109389331-e84d6a00-78ea-11eb-8752-36c2de56a5a1.png)

---

O principal fator a ser considerado para a leitura tem relação com os bits MUX3, MUX2, MUX1, MUX0

![image](https://user-images.githubusercontent.com/32770973/109389456-7590be80-78eb-11eb-9a97-c0e55b209666.png)

Como mostra a tabela, eles selecionam qual vai ser o valor que será convertido no ADC, de qual porta ou referência virá esse valor.

---

### Exemplo de código em alto nível de conversão de um canal AD

``` 
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
