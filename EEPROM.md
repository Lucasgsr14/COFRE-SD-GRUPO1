# COMO FUNCIONA A LEITURA E ESCRITA NA EEPROM

**ATENÇÃO, PARA UM CORRETO USO DA EEPROM É NECESSÁRIO QUE NÃO OCORRAM INTERRUPÇÕES DURANTE A EXECUÇÃO DAS FUNÇÕES DE LEITURA E ESCRITA**

### Registrador EECR

É o registrador responsável pelo controle da EEPROM. 

Primeiro ponto a ser destacado são os bits EEPM1:0, responsáveis pelo modo de operação da memória EEPROM

![image](https://user-images.githubusercontent.com/32770973/109494764-178be480-7a6d-11eb-9fd1-7f9b23ce1912.png)

Os modos podem ser vistos na tabela abaixo.

![image](https://user-images.githubusercontent.com/32770973/109496060-ead8cc80-7a6e-11eb-859d-fc86cb229b36.png)

---

Segundo ponto a ser destacado nesse registrador é o bit EEPE, que pode ser visto na imagem abaixo.

![image](https://user-images.githubusercontent.com/32770973/109487903-9aa83d00-7a63-11eb-885d-c0674719aea8.png)

Esse bit deve ser monitorado é o bit de enable da escrita e quando a escrita acaba esse bit assume o valor de 0 lógico, indicando assim que a função de escrita acabou.

---

Outro bit importante é o EEMPE é uma espécie de habilitador do enable das escrita EEPE. Se o EEPE for setado sem esse bit ter sido previamente setado a operação de escrita não tem nenhum efeito.

![image](https://user-images.githubusercontent.com/32770973/109494129-3ccc2300-7a6c-11eb-82ca-6c3e4b85130e.png)

Acima podemos ver o bit EEMPE. Um fator importante a ser considerado também é que a após setado o intervalo para poder setar o bit EEPE é de 4 ciclos de clock.

---

### Registrador SPMCSR

Este é o registrador que armazena o controle da memória de programa e registro de status. Nele devemos atentar para o bit SELFPRGEN.

![image](https://user-images.githubusercontent.com/32770973/109489757-1905de80-7a66-11eb-8c9d-e4440e3c896d.png)

Esse bit habilita o modo de auto programação (uma caracteristica dos microcontroladores AVR que permite que permite que o microcontrolador programa sua própria memória flash). 
Se a CPU estiver operando a flash não deve ser inserido comando de escrita ao mesmo tempo. Mas se tiver garantia que a CPU não estiver fazendo isso, essa checagem não se faz necessária.

---

### Registrador EEAR

É o registrador de endereços da EEPROM, onde são usados dois registradores de 8 bits para armazenar as partes mais significativa e menos significativa dos endereços (EEARH, EEARL).

![image](https://user-images.githubusercontent.com/32770973/109491648-9af70700-7a68-11eb-8d69-95c54921f3a3.png)

No entanto no ATMega328P o EEAR8, único bit que pode ser escrito no registrador EEARH, é inutilizado e deve sempre ser escrito com zero.

---

### Registrador EEDR

É o registrador de dados da EEPROM. Na escrita esse registrador contém os dados que serão colocados na memória. Já na leitura os dados da memória são copiados para esse registrador.

![image](https://user-images.githubusercontent.com/32770973/109492548-e958d580-7a69-11eb-9386-b51cd78c8751.png)

---

### OBS:

Para a operação de escrita é necessário levar em conta o tempo para execução dessa função que pode ser visto na tabela abaixo.

![image](https://user-images.githubusercontent.com/32770973/109496874-02648500-7a70-11eb-9c27-94636a2f80ad.png)

### Exemplo de código em C

```c
void EEPROM_write(unsigned int uiAddress, unsigned char ucData){
  /* Espera pela fim da última escrita */
  while(EECR & (1<<EEPE));
  /* Inicia os valores dos registradores de endereço e de dados */
  EEAR = uiAddress;
  EEDR = ucData;
  /* escreve 1 lógico em EEMPE para começar o processo de escrita */
  EECR |= (1<<EEMPE);
  /* escreve 1 lógico em EEPE para, de fato, iniciar o processo de escrita */
  EECR |= (1<<EEPE);
}
```

Programa de exemplo de escrita da EEPROM em linguagem C.

```c
unsigned char EEPROM_read(unsigned int uiAddress)
{
  /* Espera pela fim da última escrita */
  while(EECR & (1<<EEPE));
  /* Inicia o valor do registrador de endereço */
  EEAR = uiAddress;
  /* Escreve 1 lógico para o bit EERE, solicitando a leitura da EEPROM*/
  EECR |= (1<<EERE);
  /* Retorna o valor que estava na memória e foi copiado para o registrador EEDR */
  return EEDR;
}
```

Programa de exemplo de leitura da EEPROM em linguagem C.

---

## REFERÊNCIAS

Atmel. **8-bit AVR Microcontroller with 32K Bytes In-System Programmable Flash - DATASHEET.**. Disponível em <https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf>. Acesso em: 23 de fevereiro de 2021.
