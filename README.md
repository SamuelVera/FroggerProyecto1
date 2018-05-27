# FroggerProyecto1
# Organización del Computador/Proyecto 1/Frogger
# Código en ensamblador de MIPS MARS 4.5

	.data
pantallaAncho: .word 16
pantallaAlto: .word 10 #Alto y ancho de la pantalla Ancho 16*4 y Alto 9*4

posX: .word 8 #Posición incial de la rana en X
posY: .word 10 #Posición inicial de la rana en Y

	#Posiciones de aparición de los carros, desplazamiento a el $gp
posCar1: .word 512
posCam2: .word 508
posCar3: .word 384

	#Posiciones de memoria de la que parten los troncos, desplazamiento a el $gp
posTron1: .word 128
posTron2: .word 252
posTron3: .word 256

	#Valores hexadecimales de los colores
camion: .word 0x3F3FFF #Color azul claro para el camión
carro: .word 0x8E0CCE #Color rojo para el carro
tronco: .word 0x800000 #Color marrón para el tronco
mosca: .word 0x000000 #Color negro para la mosca
rana: .word 0x008000 #Color verde para la rana
fondoGrama: .word 0x00FF00 #Color lima para la grama
fondoAsfalto: .word 0x808080 #Color gris para el asfalto
fondoAgua: .word 0x0000FF #Color azul del agua
vida: .word 0XFF0000 #Color rojo de las vidas

pantallaDireccion: .word 0x01000000

contadores:
	.byte 20 #Se deberá reiniciar cada que llegue se suelte un carro, CONTADOR DE LA PRIMERA FILA DE CARROS
	.byte 23 #Se deberá reiniciar cada que llegue se suelte un carro, CONTADOR DE LA SEGUNDA FILA DE CARROS/CAMION
	.byte 25 #Se deberá reiniciar cada que llegue se suelte un camión, CONTADOR DE LA TERCERA FILA DE CARROS/CAMION
	.byte 21 #Se deberá reiniciar cada que llegue se suelte un tronco, CONTADOR DE LA PRIMERA FILA DE TRONCOS
	.byte 15 #Se deberá reiniciar cada que llegue se suelte un tronco, CONTADOR DE LA SEGUNDA FILA DE TRONCOS
	.byte 10 #Se deberá reiniciar cada que llegue se suelte un tronco, CONTADOR DE LA TERCERA FILA DE TRONCOS
	.byte 20 #Se deberá reiniciar cada que llegue se suelte una mosca, CONTADOR DE LA APARICIÓN DE MOSCAS
	.space 1
cont_Avanza_t: .byte 3

vidas: .word 4
score: .space 16

	#Definiciones de macros

	.macro muere($aux)
li $s1, 1
sw $t1, 0($aux)
jal reemp_move
	.end_macro
	
	.macro choque($aux) #Comprueba si choca con un carro, camión o cae al agua
add $aux, $aux, $s0
or $a0, $aux, $zero
lw $t0, 0($aux)
lw $t1, fondoAgua
beq $t0, $t1, muere_move #Camina hacia el agua
lw $t1, carro
beq $t0, $t1, muere_move #Camina hacia un carro
lw $t1, camion
beq $t0, $t1, muere_move #Camina hacia un camión
	.end_macro

	.text
	inicializar:

	#Inicialización de vidas y puntuación
li $t0, 0x00000003
sw $t0, vidas
li $t0, 0
lw $t0, score
li $s1, 0 #Confirma si se atropella o se ahoga la rana
li $s2, 0 #Confirma si gana la rana

jal pintar_Fondo

	inicial_perderVida_o_Ganar: #Inicialización con perdida de vida

jal pintar_Vidas
lw $a0, posX
lw $a1, posY
jal calculo_pos_Rana
lw $t0, rana
sw $t0, 0($v0)
or $s0, $v0, $zero


jam:
jal move_Jugador
jal Sleep

beqz $s1, jam #Si s1!=0 es que se atropelló o ahogo a la rana, hay que reinicia la posición de la rana
li $s1, 0
lw $t0, vidas
addi $t0, $t0, -1
sw $t0, vidas
bnez $t0, inicial_perderVida_o_Ganar #Mientras las vidas sean mayores a 0 se juega

###############################################################################################
#Sub ciclo de la ejecución, ventana de tiempo del jugador para moverse, de unos 0.1 segundos

	move_Jugador:

li $t0, 0

	loop_move_Jugador:

li $v0, 32
li $a0, 50 #Sleep
syscall


add $t0, $t0, $a0

li $t1, 0xffff0000 #Contiene dirección de memoria donde se almacena el bit de ready de los dispositivos I/O
lw $t2, ($t1)
beqz $t2, seguir_move_Jugador
lw $a0, 4($t1)

addi $sp, $sp, -8 #Backup de la posición de memoria y el contador en la pila
sw $ra, 0($sp)
sw $t0, 4($sp)

jal eval_ingreso_Keybord

lw $t0, 4($sp)
lw $ra, 0($sp)
addi $sp, $sp, 8
j seguir_move_Jugador


	seguir_move_Jugador:
ble $t0, 100, loop_move_Jugador

beqz $s2, salir_move_Jugador
li $s2, 0
lw $t0, score
addi $t0, $t0, 1000
sw $t0, score
j inicial_perderVida_o_Ganar

	salir_move_Jugador:
jr $ra

###############################################################################################
#Evaluar dato ingresado en el keyboard

	eval_ingreso_Keybord:
addi $sp, $sp, -4 #Backup de la posición de memoria en el ra puesto en la pila
sw $ra, 0($sp)

or $t0, $a0, $zero #Igreso por el teclado
beq $t0, 87, move_arriba
beq $t0, 119, move_arriba
beq $t0, 83, move_abajo
beq $t0, 115, move_abajo
beq $t0, 65, move_izq
beq $t0, 97, move_izq
beq $t0, 68, move_der
beq $t0, 100, move_der
j salir_eval_key

	move_arriba:
li $t2, -64
choque($t2)

lw $t1, mosca #Validar si comió una mosca
bne $t0, $t1, no_come_arriba
lw $t2, score
addi $t2, $t2, 200
sw $t2, score
	no_come_arriba:

jal reemp_move

lw $t0, rana
addi $s0, $s0, -64
sw $t0, 0($s0)
or $t1, $zero, $s0
sub $t1, $t1, $gp
srl $t1, $t1, 2 #Validar si se ha llegado al final
bge $t1, 32, salir_eval_key
lw $t2, fondoGrama
sw $t2, 0($s0)
li $s2, 1
j salir_eval_key

	move_abajo:
or $t0, $zero, $s0
sub $t0, $t0, $gp
srl $t0, $t0, 2
bge $t0, 144, salir_eval_key

li $t2, 64
choque($t2)

lw $t1, mosca #Validar si comió una mosca
bne $t0, $t1, no_come_abajo
lw $t2, score
addi $t2, $t2, 200
sw $t2, score
	no_come_abajo:
jal reemp_move
lw $t0, rana
addi $s0, $s0, 64
sw $t0, 0($s0)
j salir_eval_key

	move_izq:
or $t0, $zero, $s0
sub $t0, $t0, $gp
srl $t0, $t0, 2
li $t1, 144
	loop_comprobar_izq: #Comprobar si puede moverse a la izquierda
beq $t0, $t1, salir_eval_key
addi $t1, $t1, -16
bge $t1, 32, loop_comprobar_izq

li $t2, -4
choque($t2)

lw $t1, mosca #Comprobar si come una mosca
bne $t0, $t1, no_come_izq
lw $t2, score
addi $t2, $t2, 200 #Si come una mosca suma puntos
sw $t2, score

	no_come_izq:
jal reemp_move

lw $t0, rana
addi $s0, $s0, -4
sw $t0, 0($s0)
j salir_eval_key

	move_der:
or $t0, $zero, $s0
sub $t0, $t0, $gp
srl $t0, $t0, 2
li $t1, 159
	loop_comprobar_der: #Comprobar si se puede mover a la derecha
beq $t0, $t1, salir_eval_key
addi $t1, $t1, -16
bge $t1, 47,loop_comprobar_der

li $t2, 4
choque($t2)

lw $t1, mosca #Comprobar si come mosca
bne $t0, $t1, no_come_der
lw $t2, score
addi $t2, $t2, 200 #Si come una mosca suma puntos
sw $t2, score

	no_come_der:
jal reemp_move

lw $t0, rana
addi $s0, $s0, 4
sw $t0, 0($s0)
j salir_eval_key

	muere_move:
muere($a0)
	salir_eval_key:
lw $ra, 0($sp)
addi $sp, $sp, 4

li $t0, 0 #Limpieza de los registros
li $t1, 0
li $t2, 0
li $a0, 0

jr $ra

###############################################################################################
#Restitución de pixeles en un movimiento hacia arriba o abajo
	reemp_move:

or $t0, $s0, $zero #Posición en bytes en memoria
sub $t0, $t0, $gp #Posición en bytes con respecto al $gp
srl $t0, $t0, 2 #Posición en pixeles

bge $t0, 144, reemp_grama #Se está en una línea de grama
bge $t0, 96, reemp_asfalto #Se está en una línea de asfalto
bge $t0, 80, reemp_grama #Se está en una línea de grama

lw $t0, tronco #Se reemplaza con un tronco
sw $t0, 0($s0)
j salir_reemp_move

	reemp_asfalto: #Se reemplaza con asfalto
lw $t0, fondoAsfalto
sw $t0, 0($s0)
j salir_reemp_move

	reemp_grama: #Se reemplaza con grama
lw $t0, fondoGrama
sw $t0, 0($s0)

	salir_reemp_move:
li $t0, 0
jr $ra

###############################################################################################
#Función de sleep, complementa un ciclo de ejecución al añadirle al 0.1 segundos del jugador otros 0.1 segundos
#En a0 esta la cantidad de tiempo a pausar ejecución
	Sleep:
li $v0, 32
li $a0, 100 #100 miliseg + 100 miliseg = 0.2 segundos
syscall
li $t0, 0

addi $sp, $sp, -4 #Backup de la posición de memoria en el ra puesto en la pila
sw $ra, 0($sp)

	#Hacer operaciones con los contadores de troncos, mosca y carro/camión
	loopContadoresSleep:
lbu $a2, contadores($t0) #Se carga el contador de memoria
addi $a2, $a2, 1 #Se le suma uno
or $a0, $t0, $zero #Se mueve el apuntador de t0 a a0
jal generarCCT #Se llama la función de generar carro, 
sb $a2, contadores($t0)
addi $t0, $t0, 1
ble $t0, 6, loopContadoresSleep

jal moverCCT

jal dibujarTodo

li $t0, 0 #Limpiar registros
li $a0, 0
li $a1, 0
li $a2, 0

lw $ra, 0($sp)
addi $sp, $sp, 4

	salir_Sleep:
jr $ra

###############################################################################################
#Procedimiento para generar los objetos dañinos para la rana
#Se tiene en a2 el contador de carro, camion o tronco
#En a0 esta la posición del contador 
#Se efectua movimientos dependiendo de sus posiciones en memo
	generarCCT:

blt $a2, 8, seguir #Si el contador es menor a 25 no hay razón para agregar un carro, camión o tronco
or $t0, $a0, $zero
li $v0, 30 #Preparación del seed con el tiempo del sistema
syscall
or $a1, $a0, $zero
li $v0, 40
syscall
li $a0, 40
li $a1, 100
li $v0, 42
syscall
blt $a0, 90, seguir #Si el pseudorandom es menor a 70 no se genera carro/tronco/camion (30% de las veces se genera)
addi $sp, $sp, -4 #Guardar contenido del registro ra en el stack

sw $ra, 0($sp)

lw $a1, pantallaAncho
lw $a2, pantallaAlto
beq $t0, 0, filaUnoCarro
beq $t0, 1, filaDosCarro
beq $t0, 2, filaTresCarro
beq $t0, 3, filaUnoTronco
beq $t0, 4, filaDosTronco
beq $t0, 5, filaTresTronco

jal poner_Mosca

	filaTresTronco:
lw $t1, tronco
lw $t2, posTron3
addu $t2, $t2, $gp
lw $t3, 0($t2)
lw $t4, 4($t2)
lw $t5, 8($t2)
beq $t3, $t1, restaura
beq $t4, $t1, restaura
beq $t5, $t1, restaura
sw $t1, 0($t2)
j restaura

	filaDosTronco:
lw $t1, tronco
lw $t2, posTron2
addu $t2, $t2, $gp
lw $t3, 0($t2)
lw $t4, -4($t2)
lw $t5, -8($t2)
beq $t3, $t1, restaura
beq $t4, $t1, restaura
beq $t5, $t1, restaura
sw $t1, 0($t2)
j restaura

	filaUnoTronco:
lw $t1, tronco
lw $t2, posTron1
addu $t2, $t2, $gp
lw $t3, 0($t2)
lw $t4, 4($t2)
lw $t5, 8($t2)
beq $t3, $t1, restaura
beq $t4, $t1, restaura
beq $t5, $t1, restaura
sw $t1, 0($t2)
j restaura
	
	filaTresCarro:
lw $t1, carro
lw $t2, posCar3
addu $t2, $t2, $gp
lw $t3, 0($t2)
lw $t4, 4($t2)
beq $t3, $t1, restaura
beq $t4, $t1, restaura
sw $t1, 0($t2)
j restaura

	filaDosCarro:
lw $t1, camion
lw $t2, posCam2
addu $t2, $t2, $gp
lw $t3, 0($t2)
lw $t4, -4($t2)
lw $t5, -8($t2)
beq $t3, $t1, restaura
beq $t4, $t1, restaura
beq $t5, $t1, restaura
sw $t1, 0($t2)
j restaura

	filaUnoCarro:
lw $t1, carro
lw $t2, posCar1
addu $t2, $t2, $gp
lw $t3, 0($t2)
lw $t4, 4($t2)
beq $t3, $t1, restaura
beq $t4, $t1, restaura
sw $t1, 0($t2)

	restaura:

lw $ra, 0($sp) #Restaurar posición de memoria
addi $sp, $sp, 4
li $a2, 0

	seguir:
jr $ra

###############################################################################################
#Procedimiento para poner una mosca
	poner_Mosca:

lw $t3, carro
lw $t4, camion #Se usan para no generar una mosca sobre un carro, tronco o camión
lw $t5, tronco
lw $t6, rana

li $v0, 30 #Preparación del seed con el tiempo del sistema
syscall
or $a1, $a0, $zero
li $v0, 40
syscall
li $a0, 40 #Preparación del pseudorandom
li $a1, 100
li $v0, 42
syscall
or $t0, $zero, $a0

ble $t0, 70, seguir_poner_Mosca
	
	loop_poner_Mosca:
li $a0, 40 #Preparación del pseudorandom para posición
li $a1, 144 #Pixeles para la posición posible de la mosca 
li $v0, 42
syscall
ble $a0, 31, loop_poner_Mosca
or $t1, $a0, $zero
sll $t1, $t1, 2 #Posición en bytes
addu $t1, $t1, $gp #Relativo al $gp
lw $t7, 0($t1)
beq $t7, $t3, loop_poner_Mosca
beq $t7, $t4, loop_poner_Mosca #Revisar que no haya carro, camión, tronco o rana
beq $t7, $t5, loop_poner_Mosca
beq $t7, $t6, loop_poner_Mosca

lw $t0, mosca
sw $t0, 0($t1) #Se guarda la mosca

	seguir_poner_Mosca:

li $t0, 0 #Limpiar registros
li $t1, 0
li $t2, 0
li $t3, 0
li $t4, 0
li $t5, 0
li $t6, 0
li $t7, 0
li $a0, 0

jr $ra


###############################################################################################
#Procedimiento para el movimiento de los carros, camiones y troncos

	moverCCT:

lbu $t0, cont_Avanza_t
addiu $t0, $t0, 1

addi $sp, $sp, -8
sw $ra, 0($sp)
sw $t0, 4($sp)


ble $t0, 5, seguirCCT1
	#Preparar para movimiento de izquierda a derecha

li $a0, 0 #t0
li $a1, 188 #t1
addu $a1, $a1, $gp
jal moverIzq_Der_t #Fila 1

li $a0, 0
li $a1, 192
addu $a1, $a1, $gp
lw $a2, tronco
lw $a3, fondoAgua
jal moverDer_Izq #Fila 2

li $a0, 0 #t0
li $a1, 316 #t1
addu $a1, $a1, $gp
jal moverIzq_Der_t #Fila 3

li $t0, 0 #Reinicia contador de movimiento
sw $t0, 4($sp)

	seguirCCT1:

li $a0, 0
li $a1, 444
addu $a1, $a1, $gp
jal moverIzq_Der_c #Fila 3 de carros

li $a0, 0
li $a1, 448
addu $a1, $a1, $gp
lw $a2, camion
lw $a3, fondoAsfalto
jal moverDer_Izq #Fila 2 de camion

li $a0, 0
li $a1, 572
addu $a1, $a1, $gp
jal moverIzq_Der_c #Fila 1 de carros

lw $t0, 4($sp)	
lw $ra, 0($sp)
addi $sp, $sp, 8
sb $t0, cont_Avanza_t
jr $ra

###############################################################################################
#Movimineto de izquierda a derecha de troncos
#$t0 es contador de iteraciones, $t1 y $t2 apuntadores de memoria
#$t3 y $t4 ayudan para el intercambio de datos en memoria

	moverIzq_Der_t:
addi $sp, $sp, -4
sw $ra, 0($sp)

or $t0, $zero, $a0
or $t1, $zero, $a1
lw $t2, tronco
lw $t3, fondoAgua

	loop_mover_i_d_t: #Se toman en cuenta 6 casos para correr de izquierda a derecha los troncos
			  #El barrido se hace de derecha a izquierda

lw $t4, 0($t1)
lw $t5, -4($t1)
lw $t6, -8($t1)
lw $t7, -12($t1)
                                                                      #Es 1 o 0 espacios de tronco?
bne $t4, $t2, sigue_mover_i_d_t    #Si el actual no es tronco se busca el siguiente
                #De lo contrario se evalua el contenido del anterior

beq $t0, 60, ext_i2_i_d_t        #De lo contrario se evalua de que extremo de la fila está el espacio de tronco
beq $t5, $t2, cond2_mover_i_d_t #Es 1 o 2 espacios de tronco?

sw $t3, 0($t1) #Si está del lado derecho se agrega un agua en la posición actual
j sigue_mover_i_d_t

    ext_i2_i_d_t:            #Si está del lado izquierdo se agrega un tronco a la derecha
sw $t2, 4($t1)
j sigue_mover_i_d_t        #Se salta al siguiente

    cond2_mover_i_d_t:    #Son 3 o 2 espacios contiguos de tronco?
			#De lo contrario se evalua de que extremo se encuentra
beq $t0, 56, ext_i3_i_d_t
beq $t6, $t2, cond3_mover_i_d_t    #Si los 3 son troncos se evalua la siguiente condición

sw $t3, -4($t1)           #Si está del lado izquierdo se agrega un tronco a la derecha
j sigue_mover_i_d_t

    ext_i3_i_d_t:            #Si está del lado derecho se agrega un agua en la posición a -4 de la actual
sw $t2, 4($t1)
j sigue_mover_i_d_t

    cond3_mover_i_d_t:    #Extremo en el que se ubica el tronco o si está en el centro
bne $t0, 0, ext_if_i_d_t        #Si el iterador t0 es igual a 0 no han habido desplazamiento y se encuentra en el extremo derecho
                	    #Si está en el extremo derecho se agrega agua en la posición -8 a la actual
sw $t3, -8($t0)
j sigue_mover_i_d_t

    ext_if_i_d_t:            #De lo contrario está en el centro o a la izquierda, se desplaza todo el tronco

sw $t2, 4($t1)
sw $t2, 0($t1)
sw $t2, -4($t1)
sw $t3, -8($t1)
addi $t0, $t0, 4
addi $t1, $t1, -4
j sigue_mover_i_d_t

	sigue_mover_i_d_t:
addi $t0, $t0, 4
addi $t1, $t1, -4

ble $t0, 60, loop_mover_i_d_t

lw $ra, 0($sp)
addi $sp, $sp, 4
jr $ra

###############################################################################################
#Movimineto de izquierda a derecha de carros
#Mismo concepto del movimiento de troncos pero con parámetros distintos
	moverIzq_Der_c:
addi $sp, $sp, -4
sw $ra, 0($sp)

or $t0, $zero, $a0
or $t1, $zero, $a1
lw $t2, carro
lw $t3, fondoAsfalto

	loop_mover_i_d_c: #Se toman en cuenta los casos para correr de izquierda a derecha los carros
			  #El barrido se hace de derecha a izquierda

lw $t4, 0($t1)
lw $t5, -4($t1)
                                   #Es 1 o 0 espacios de carro?
bne $t4, $t2, sigue_mover_i_d_c    #Si el actual no es carro se busca el siguiente
                #De lo contrario se evalua el contenido del anterior

beq $t0, 60, ext_i1_i_d_c       #Se estamos en la última iteración y hay un espacio de carro confirmado se procede al primer caso
beq $t5, $t2, cond2_mover_i_d_c #Es 2 espacios de carro?

sw $t3, 0($t1) #Si está del lado derecho se agrega un espacio de asfalto en la posición actual
j sigue_mover_i_d_c

    ext_i1_i_d_c:  #Si está del lado izquierdo y es un solo espacio, se agrega un carro a la derecha
lw $t4, 4($t1)
lw $t5, rana
bne $t4, $t5, seguir_i1_i_d_c #Verificar si se atropella la rana
li $s1, 1
	seguir_i1_i_d_c:
sw $t2, 4($t1)

j sigue_mover_i_d_c       #Se salta al siguiente

    cond2_mover_i_d_c:    #Son 2 espacios de carro
bne $t0, 0, ext_i2_i_d_c  #Se confirma si están del lado izquierdo o en otro lugar

sw $t3, -4($t1)           #Si está del lado derecho se llena de asfalto la posición -4 
j sigue_mover_i_d_c

    ext_i2_i_d_c:            #Si está del lado izquierdo o en el centro se desplaza el carro
sw $t3, -4($t1)
sw $t2, 0($t1)

lw $t4, 4($t1)
lw $t5, rana
bne $t4, $t5, seguir_i2_i_d_c #Verificar si se atropella la rana
li $s1, 1

	seguir_i2_i_d_c:

sw $t2, 4($t1)
addi $t0, $t0, 4
addi $t1, $t1, -4

	sigue_mover_i_d_c:
addi $t0, $t0, 4

addi $t1, $t1, -4

ble $t0, 60, loop_mover_i_d_c

lw $ra, 0($sp)
addi $sp, $sp, 4

jr $ra

###############################################################################################
#Movimiento de derecha a izquierda de camiones y troncos
#Función general para ambos pues miden 3 pixeles

moverDer_Izq:
addi $sp, $sp, -4
sw $ra, 0($sp)

or $t0, $zero, $a0
or $t1, $zero, $a1
or $t2, $zero, $a2
or $t3, $zero, $a3

	loop_mover_d_i: #Se toman en cuenta 6 casos para correr de derecha a izquierda
			  #El barrido se hace de izquierda a derecha

lw $t4, 0($t1)
lw $t5, 4($t1)
lw $t6, 8($t1)
lw $t7, 12($t1)
                                                                      #Es 1 o 0 espacios de tronco?
bne $t4, $t2, sigue_mover_d_i    #Si el actual no es tronco se busca el siguiente
                #De lo contrario se evalua el contenido del anterior

beq $t0, 60, ext_i2_d_i        #De lo contrario se evalua de que extremo de la fila está el espacio lleno
beq $t5, $t2, cond2_mover_d_i #Es 1 o 2 espacios llenos?

sw $t3, 0($t1) #Si está del lado izquierdo se agrega un fondo en la posición actual
j sigue_mover_d_i

    ext_i2_d_i:            #Si está del lado derecho se agrega un contenido a la izquierda
lw $t4, -4($t1)
lw $t5, rana
bne $t4, $t5, seguir_i1_d_i #Verificar si se atropella la rana
li $s1, 1
	seguir_i1_d_i:
sw $t2, -4($t1)
j sigue_mover_d_i        #Se salta al siguiente

    cond2_mover_d_i:    #Son 3 o 2 espacios contiguos llenos?
			#De lo contrario se evalua de que extremo se encuentra
beq $t0, 56, ext_i3_d_i
beq $t6, $t2, cond3_mover_d_i    #Si los 3 estan llenos se evalua la siguiente condición

sw $t3, 4($t1)           #Si está del lado izquierdo se agrega un vacio a la derecha
j sigue_mover_d_i

    ext_i3_d_i:         #Si está del lado derecha se agrega un elemento en la posición a -4 de la actual
lw $t4, -4($t1)
lw $t5, rana
bne $t4, $t5, seguir_21_d_i #Verificar si se atropella la rana
li $s1, 1
	seguir_21_d_i:
sw $t2, -4($t1)
j sigue_mover_d_i

    cond3_mover_d_i:    #Extremo en el que se ubica el tronco o si está en el centro
bne $t0, 0, ext_if_d_i      #Si el iterador t0 es igual a 0 no han habido desplazamiento y se encuentra en el extremo izquierdo
                	    #Si está en el extremo izquierdo se agrega fondo en la posición 8 a la actual
sw $t3, 8($t1)
j sigue_mover_d_i

    ext_if_d_i:            #De lo contrario está en el centro o a la derecha, se desplaza todo el elemento

lw $t4, -4($t1)
sw $t2, -4($t1)
sw $t2, 0($t1)
sw $t2, 4($t1)
sw $t3, 8($t1)
addi $t0, $t0, 4
addi $t1, $t1, 4


lw $t5, rana
bne $t4, $t5, seguir_3_d_i #Verificar si se atropella la rana
li $s1, 1
	seguir_3_d_i:
j sigue_mover_d_i

	sigue_mover_d_i:
addi $t0, $t0, 4
addi $t1, $t1, 4

ble $t0, 60, loop_mover_d_i

lw $ra, 0($sp)
addi $sp, $sp, 4

jr $ra

###############################################################################################
#Imprimir toda el bitmap guardado
	dibujarTodo:
addi $sp, $sp, -4
sw $ra, 0($sp)

or $t0, $zero, $gp
lw $a1, pantallaAncho
lw $a2, pantallaAlto

	loopDibujarTodo: #Iterar por todas las posiciones de memoria a para pintarlas
lw $t1, 0($t0)
or $a0, $zero, $t1
or $a1, $zero, $t0
jal dibujar
addi $t0, $t0, 4
ble $t0, 636, loopDibujarTodo

lw $ra, 0($sp)
addi $sp, $sp, 4
jr $ra

###############################################################################################
#Pintar vidas
#Combo Sencillo
	pintar_Vidas:

lw $t0, vidas
li $t1, 0
lw $t2, vida
addu $t1, $t1, $gp #Apuntador a la memoria
	loop_pintar_Vidas:
sw $t2, 0($t1)
addi $t1, $t1, 4
addi $t0, $t0, -1
bnez $t0, loop_pintar_Vidas
li $t0, 0x00000000
sw $t0, 0($t1)

jr $ra

###############################################################################################
#Pintar fondos
#Balurdo y sencillo
	pintar_Fondo:
lw $t5, pantallaAncho
lw $t0, pantallaAlto
multu $t5, $t0
mflo $t5
sll $t5, $t5, 2
addu $t5, $t5, $gp #Posición relativa al global poiner

	#a1 contiene grama, a2 asfalto y a3 agua
lw $t0, fondoGrama
lw $t1, fondoAsfalto
lw $t2, fondoAgua
or $t3, $zero, $gp #t0 sirve de puntero
addi $t3, $t3, 64 #Se obvia la primera fila con las vidas
addi $t4, $t3, 64 #Comparación para alternar entre secciones
	loopPintarSeccion1: #Sección de grama
sw $t0, 0($t3)
addi $t3, $t3, 4
blt $t3, $t4, loopPintarSeccion1
addi $t4, $t4, 192
	loopPintarSeccion2: #Sección de agua
sw $t2, 0($t3)
addi $t3, $t3, 4
blt  $t3, $t4, loopPintarSeccion2
addi $t4, $t4, 64
	loopPintarSeccion3: #Seccion de grama 2
sw $t0, 0($t3)
addi $t3, $t3, 4
blt  $t3, $t4, loopPintarSeccion3
addi $t4, $t4, 192
	loopPintarSeccion4: #Seccion de asfalto
sw $t1, 0($t3)
addi $t3, $t3, 4
blt $t3, $t4, loopPintarSeccion4
	loopPintarSeccion5: #Seccion de grama 3
sw $t0, 0($t3)
addi $t3, $t3, 4
blt $t3, $t5, loopPintarSeccion5
li $t3, 0
li $t4, 0 #Limpiar registros
jr $ra

###############################################################################################
#Calculo de la posición como dirección de memoria (Bytes)
#Se pasan posX en $a0 y posY en $a1
#Se retorna en $v0 la dirección relativa
	calculo_pos_Rana:
or $v0, $a0, $zero #v0 = posX
lw $a0, pantallaAncho #a0 = 16
addi $a1, $a1, -1 #a1 = a1 - 1
multu $a0, $a1 
mflo $a0 #a0 = a1*16
addu $v0, $v0, $a0 #Posición en pixeles
sll $v0, $v0, 2 #Multiplicar por 4 para transportar a bytes
addu $v0, $v0, $gp #Se suma la posición del global pointer
jr $ra

###############################################################################################
#Se tiene en $a1 el ancho de pantalla, el alto en $a2 y en $a3 la posicion a dibujar
#Se tiene en $a0 el color a dibujar en el bitmap
#Solo funciona con un pixel único
	dibujar:
lw $a1, pantallaAncho
lw $a2, pantallaAlto
multu $a1, $a2 #Total de pixeles de la pantalla
mflo $a1
sll $a1, $a1, 2 #Total de bytes en la pantalla
addu $a1, $a1, $gp #Relativo al global pointer
or $a2, $zero, $gp #En el a2 se guarda la posición 0 relativa de la pantalla
	cicloDibujar:
bne $a2, $a3, siguienteDibujar
sw $a0, ($a2)
j salDibujar
	siguienteDibujar:
addi $a2, $a2, 4
ble $a2, $a1, cicloDibujar
	salDibujar:
jr $ra
