```


start:
	ldi r16, (1 << REFS0)  ; setando referencia de conversão e referencia ADC4
	sts ADMUX, r16						 ;
	ldi r17, (1 << ADPS1) | (1 << ADPS0)  ; divisão por 8
	sts ADCSRA, r17	 
	ori r17, (1 << ADEN) 
	sts ADCSRA, r17
	ori r17, (1 << ADSC) 
	sts ADCSRA, r17
	rcall clear_registradores
loop:
	lds r20, ADCSRA
	sbrc r20, ADIF
	rjmp continue
	rjmp loop
continue:
	sts ADCSRA, r17
	lds r27, ADCH
	lds r28, ADCL
	ldi r25, 10
	ldi r26, 1
	ldi r29, 255
	ldi r30, 0xFF
	ldi r31, 0x03
	rcall clear_registradores

conversao_999:
	ldi r24, 0x00
	cpse r27, r17 ; LOW(ADC), LOW(GERAL)
	adiw r24, 1
	cpse r28, r23 ; HIGH(ADC), HIGH(GERAL)
	adiw r24, 1
    cpi r24, 0
	breq final
	rcall limitador
	cpse r17, r29 ; LOW(GERAL), 255
	rjmp soma_low		  
	rjmp soma_high
soma:
	add r18, r26 ; LOCAL, 1
	mov r19, r18  ;	UNIDADES, LOCAL
	cpse r18, r25 ; LOCAL, 10
	rjmp conversao_999
	rcall soma_dezena
	cpse r20, r25 ; DEZENAS, 10
	rjmp conversao_999
	rcall soma_centena
	rjmp conversao_999

limitador:
	ldi r24, 0x00
	cpse r17, r30 ; LOW(GERAL), 0xFF
	adiw r24, 1
	cpse r23, r31 ; HIGH(GERAL), 0x03
	adiw r24, 1
	cpi r24, 0
	breq final
	ret

soma_dezena:
	clr r18		 ; LOCAL
	clr r19		 ; UNIDADES
	add r20, r26 ; DEZENAS, 1
	ret

soma_centena:
	clr r20		 ; DEZENAS
	add r21, r26; CENTENAS, 1
	ret

final:
	ldi r25, 1

clear_registradores:
	clr r16
	clr r17
	clr r18
	clr r19
	clr r20
	clr r21
	clr r22
	clr r23
	ret

soma_low:
	add r17, r26
	rjmp soma

soma_high:
	ldi r17, 0x00
	add r23, r26
	rjmp soma

```
