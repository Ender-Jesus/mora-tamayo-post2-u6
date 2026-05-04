# lab6\_pos2

laboratorio post2 unidad 6



================================================================

&#x20; lab6\_modos.asm — MODOS DE DIRECCIONAMIENTO x86

&#x20; Unidad 6 Post-Contenido 2 | Arquitectura de Computadores

================================================================





========================================

MODO 1: INMEDIATO

========================================



Fórmula EA:

&#x20; No aplica. El dato esta embebido directamente en el opcode.

&#x20; No se accede a memoria de datos.



Instrucciones usadas:

&#x20; MOV ax, 100      ; AX = 0064h

&#x20; MOV bx, 0A5h     ; BX = 00A5h

&#x20; ADD cx, 55       ; CX += 0037h

&#x20; AND dx, 00FFh    ; mascara inmediata



Valores observados en DEBUG:

&#x20; AX = 0064h  (100 decimal)

&#x20; BX = 00A5h  (165 decimal)

&#x20; El valor aparece visible dentro del opcode al desensamblar.



Comando DEBUG:

&#x20; U 0100

&#x20; -> Observar: "MOV AX,0064" — el 64h esta en el opcode





\----------------------------------------





========================================

MODO 2: DIRECTO

========================================



Fórmula EA:

&#x20; EA = direccion\_simbolo

&#x20; Direccion fija codificada en la instruccion.

&#x20; Calculada en ensamblado como: 100h + offset del simbolo.



Instrucciones usadas:

&#x20; MOV ax, \[var\_x]       ; AX = 0FFFFh

&#x20; MOV bx, \[array]       ; BX = 000Ah  (primer elemento = 10)

&#x20; MOV cx, \[nota1]       ; CX = 0055h  (85 decimal)

&#x20; MOV \[var\_x], word 0   ; escribe 0 directamente en memoria



Valores observados en DEBUG:

&#x20; AX = FFFFh   (var\_x)

&#x20; BX = 000Ah   (array\[0] = 10)

&#x20; CX = 0055h   (nota1 = 85)



&#x20; Dump del array en memoria (little-endian):

&#x20; 0A 00 | 14 00 | 1E 00 | 28 00 | 32 00

&#x20;  10       20      30      40      50



Comando DEBUG:

&#x20; D DS:0103

&#x20; -> Ver bytes: 0A 00  14 00  1E 00  28 00  32 00





\----------------------------------------





========================================

MODO 3: INDIRECTO POR REGISTRO

========================================



Fórmula EA:

&#x20; EA = \[SI]  o  \[BX]

&#x20; El registro contiene la direccion del operando.

&#x20; La direccion puede cambiar en tiempo de ejecucion (puntero).



Instrucciones usadas:

&#x20; MOV si, nota1    ; SI = direccion de nota1 (no su valor)

&#x20; MOV ax, \[si]     ; AX = mem\[SI] = 0055h

&#x20; MOV si, nota2    ; SI = direccion de nota2

&#x20; MOV bx, \[si]     ; BX = mem\[SI] = 0049h

&#x20; ADD ax, bx       ; AX = 85 + 73 = 158  (009Eh)

&#x20; SHR ax, 1        ; AX = 158 / 2 = 79   (004Fh)

&#x20; MOV si, promedio ; SI = direccion de promedio

&#x20; MOV \[si], ax     ; mem\[SI] = 79  (guarda via puntero)



Valores observados en DEBUG:



&#x20; Instruccion          | Registro | Valor

&#x20; ---------------------|----------|------------------

&#x20; MOV si, nota1        | SI       | dir(nota1)

&#x20; MOV ax, \[si]         | AX       | 0055h  (85 dec)

&#x20; MOV si, nota2        | SI       | dir(nota2)

&#x20; MOV bx, \[si]         | BX       | 0049h  (73 dec)

&#x20; ADD ax, bx           | AX       | 009Eh  (158 dec)

&#x20; SHR ax, 1            | AX       | 004Fh  (79 dec)

&#x20; MOV si, promedio     | SI       | dir(promedio)

&#x20; MOV \[si], ax         | mem\[SI]  | 004Fh  (79 dec)



Comando DEBUG:

&#x20; T  (traza paso a paso — observar como SI cambia)

&#x20; D DS:dir\_promedio

&#x20; -> Verificar que aparece: 4F 00





\----------------------------------------





========================================

MODO 4: INDEXADO  (Base + Indice + Desplazamiento)

========================================



Fórmula EA:

&#x20; EA = BX + SI

&#x20; EA = BX + SI + desplazamiento

&#x20; Base fija en BX + indice variable en SI + desplazamiento constante.

&#x20; Ideal para recorrer arrays y acceder campos de structs.



Instrucciones usadas:

&#x20; ; Acceso a elemento especifico: array\[2] = 30

&#x20; MOV bx, array     ; BX = direccion base del array

&#x20; MOV si, 4         ; SI = 2 \* 2 bytes = indice 2

&#x20; MOV ax, \[bx+si]   ; AX = array\[2] = 001Eh (30)



&#x20; ; Suma de todos los elementos (bucle)

&#x20; XOR ax, ax        ; AX = 0 (acumulador)

&#x20; MOV bx, array     ; BX = base

&#x20; MOV cx, 5         ; CX = 5 iteraciones

&#x20; XOR si, si        ; SI = 0

&#x20; .bucle:

&#x20;   ADD ax, \[bx+si] ; AX += elemento actual

&#x20;   ADD si, 2       ; avanzar 2 bytes (word)

&#x20;   LOOP .bucle

&#x20; ; resultado: AX = 10+20+30+40+50 = 150



&#x20; ; Acceso a campos del struct con desplazamiento fijo

&#x20; MOV bx, nota1     ; BX = base del struct

&#x20; MOV ax, \[bx]      ; AX = nota1 = 85   (offset 0)

&#x20; MOV cx, \[bx+2]    ; CX = nota2 = 73   (offset 2)

&#x20; MOV dx, \[bx+4]    ; DX = promedio = 79 (offset 4)



Valores observados en DEBUG:

&#x20; AX = 001Eh   (array\[2] = 30)

&#x20; AX = 0096h   (suma total = 150 decimal)

&#x20; AX = 0055h   (nota1 = 85)

&#x20; CX = 0049h   (nota2 = 73)

&#x20; DX = 004Fh   (promedio = 79)



Comando DEBUG:

&#x20; T  (trazar el bucle instruccion por instruccion)

&#x20; R AX

&#x20; -> Al salir del bucle: AX = 0096h  (150 decimal)





================================================================

CHECKPOINTS

================================================================



CHECKPOINT 1 — Dump en memoria

&#x20; Compilar : nasm -f bin lab6\_modos.asm -o lab6\_modos.com

&#x20; Abrir    : debug lab6\_modos.com

&#x20; Comando  : D DS:100



&#x20; Valores esperados (little-endian):

&#x20;   array  ->  0A 00  14 00  1E 00  28 00  32 00

&#x20;   nota1  ->  55 00  (85)

&#x20;   nota2  ->  49 00  (73)



CHECKPOINT 2 — Trazado modo indirecto

&#x20; Ver tabla de instrucciones en MODO 3 (arriba).

&#x20; Confirmar AX = 0055h despues de "MOV ax, \[si]".



CHECKPOINT 3 — Modo indexado y recorrido inverso

&#x20; Bucle original  : AX = 0096h  (150) al finalizar

&#x20; Recorrido inverso (SI inicia en 8, decrementa 2 por iteracion):

&#x20; AX = 0096h  (150) — resultado identico



================================================================

COMMITS

================================================================



&#x20; feat: modos inmediato y directo

&#x20; feat: modo indirecto

&#x20; feat: recorrido inverso array



================================================================

