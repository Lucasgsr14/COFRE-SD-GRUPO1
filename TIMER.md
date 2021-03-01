# COMO FUNCIONA O TIMER 16 BITS NO ATMEGA328P

## O que tem que ser manipulado para seu correto funcionamento

### Registrador TCCR1A/TCCR1B

Primeiro registrador a ser abordado é o TCCR1A/TCCR1B, que controlam qual modo será usado no timer, a comparação utilizada, etc.

![image](https://user-images.githubusercontent.com/32770973/109468083-dda9e680-7a4a-11eb-9936-93fc36c4352d.png)

![image](https://user-images.githubusercontent.com/32770973/109467552-18f7e580-7a4a-11eb-91a8-d871a4c1f5d5.png)

Para esse registrador vamos abordar primeiramente os bits WGM12:10, responsáveis pelo controle do modo de operação como podemos ver na tabela abaixo.

![image](https://user-images.githubusercontent.com/32770973/109468624-a12aba80-7a4b-11eb-93be-ac31a1c6d75d.png)

No nosso trabalho optamos por usar o modo CTC (Clear Timer on Compare Match) que basicamente reseta o contador quando uma comparação feita com um número definido no registrador OCR1A/OCR1B (registradores de 8 bits que formam um número de 16 bits) é verdadeira, disparando um evento de interrupção.

Para selecionar esse modo devemos por o bit WGM12 em 1 e os demais bits do seletor WGM para 0 conforme mostrado na imagem abaixo.

![image](https://user-images.githubusercontent.com/32770973/109469645-1054de80-7a4d-11eb-8f12-5e20f8272215.png)

Mudança essa que será feita especificamente no registrador TCCR1B, ja que os valores iniciais dos bits são zero.

![image](https://user-images.githubusercontent.com/32770973/109469908-72154880-7a4d-11eb-84af-e06f099905e4.png)

---

Ainda sobre esses registradores um ponto importante que deve ser levado em consideração são os bits CS12:10 que são os bits de seleção de clock para o time, se ele vai usar o clock geral utilizado, no caso do nosso projeto um clock de alimentação externa de 10,24 MHz, ou vai fazer uma divisão desse clock por um valor tabelado, como pode ser observado abaixo.

![image](https://user-images.githubusercontent.com/32770973/109471691-e224ce00-7a4f-11eb-8bb7-59c4682fae8d.png)

em nosso projeto optamos pela divisão do clock em 1024, que nos permite, de acordo com o clock geral escolhido, ter um multiplo de contagem, dentro dos 16 bits de tamanho total, que seja possível de contar 0,5 segundos, o tempo exato definido para salvar as informações na memória do microcontrolador para posterior comparação.

![image](https://user-images.githubusercontent.com/32770973/109472624-1fd62680-7a51-11eb-82ec-6e816ea452e1.png)

Dessa maneira, como pode ser visto na tabela dos seletores acima, temos que alterar os bits CS12 e CS10 para 1.

**Um informação muito importante é que em código essa deve ser a instrução dada apenas no momento em que todos os outros registradores já foram devidamente modificados, pois ela que da início a contagem do Timer.** 

---

### Registrador OCR1A/OCR1B

Estes registradores são responsáveis por guardar o número de comparação do Timer, em nosso caso mais especificamente do modo CTC. São dois registradores de 16 bits (na realidade 2 registradores de 8 bits para cada) que podem ser usados de acordo com o modo selecionado, mas em nosso caso o registrador de referência é o OCR1A, visto na imagem abaixo.

![image](https://user-images.githubusercontent.com/32770973/109474660-7fcdcc80-7a53-11eb-881e-5a1211cbbae8.png)

---

### Registrador TIMSK1

Este registrador é responsável por, entre outras funções, simbolizar de onde está vindo a referência para a comparação. Em nosso caso específico está vindo do registrador OCR1A, como visto no tópico anterior.

![image](https://user-images.githubusercontent.com/32770973/109475667-bf48e880-7a54-11eb-9575-6196afa559f2.png)

Como visto acima o bit que necessita de mudança é o OCIE1A.

---

### Calculadora de Timer AVR

Para facilitar os cálculos utilizamos a [AVR Timer Calculator](https://eleccelerator.com/avr-timer-calculator/) que leva em conta as tabelas disponíveis, em nosso caso no datasheet do ATMega328P, e torna mais fácil setar a interrupção.

### Exemplo de código em C

```c
#include <avr/io.h>
#include <avr/interrupt.h>

include main(void){
	DDRB = 0x01;                //Habilitando a porta: PORTB0
	TCCR1B = (1 << WGM12);      //Setando o modo CTC
	OCR1A = 5000;               //Colocando o número de ticks até a interrupção ser efetuada
	TIMSK1 = (1 << OCIE1A);     //Referenciando o registrador zero para comparação
	sei();                      //Setando o evento de interrupção
	TCCR1B |= (1 << CS12) | (1 << CS10);    //Setando o a divisão de clock por 1024
}
ISR(TIMER1_COMPA_vect){       //Definindo o evento de interrupção
	PORTB ^= (1 << PORTB0);     //Alternando o valor da porta PORTB0 a cada interrupção gerada pelo timer
}
```

Programa de exemplo de utilização do Timer 16 bits do ATMega328P em linguagem C.

## REFERÊNCIAS

humanHardDrive. **Learning AVR-C Episode 6: Timers**. Youtube, 21 de dezembro de 2012. Disponível em <https://www.youtube.com/watch?v=cAui6116XKc&ab_channel=humanHardDrive>. Acesso em: 27 de fevereiro de 2021.
Atmel. **8-bit AVR Microcontroller with 32K Bytes In-System Programmable Flash - DATASHEET.**. Disponível em <https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf>. Acesso em: 23 de fevereiro de 2021.

