# COMO FUNCIONA O PWM NO ATMEGA328P

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

---

### Registrador OCR0A

![image](https://user-images.githubusercontent.com/32770973/109575011-b3e6d300-7acf-11eb-8fe2-a153b36bd404.png)

Neste registrador guardamos um número para comparação, no caso vai ser em cima desse número de 8 bits que o gerador PWM vai modular a onda para valores de tensão
variando e 19,61 mV para cada unidade de 0 até 255.

---

### Registrador DDRC 

![image](https://user-images.githubusercontent.com/32770973/109575539-c7df0480-7ad0-11eb-85f4-3cce4ef27cf8.png)

Por fim esse registrador é o último que abordaremos que serve para setar a porta D6 que está associada a saida desse modo PWM. Basta escrever 1 lógico no bit DDD6, onde mostra a tabela abaixo:

![image](https://user-images.githubusercontent.com/32770973/109575781-420f8900-7ad1-11eb-9e11-82cb3148ac54.png)


### Exemplo de código em C

```c
void geraPWM(int num){
	TCCR0B |= (1 << CS00) | (1 << CS01); //prescale de divisão por 64
	TCCR0A |= (1 << WGM01) | (1 << WGM00) | (1 << COM0A1); //modo fast pwm, não invertido
	DDRC |= (1 << DDD6); //setando a porta D6 como saída
	OCR0A = num; //armazena o número de comparaçãoD
}
```

Programa para fazer uma conversão AD.

## REFERÊNCIAS

MAITI, Sayantan. **RGB LED with Atmega328 AVR using PWM | Datasheet, C Code | Explained in details**. Youtube, 11 out. de 2020. Disponível em <https://www.youtube.com/watch?v=IUW0OTU7BHM&ab_channel=SayantanMaiti>. Acesso em: 01 de fevereiro de 2021.
Atmel. **8-bit AVR Microcontroller with 32K Bytes In-System Programmable Flash - DATASHEET.**. Disponível em <https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf>. Acesso em: 23 de fevereiro de 2021.
