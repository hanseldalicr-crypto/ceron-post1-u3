# ceron-post1-u3
Laboratorio - Manejo del DEBUG: Exploración, Ensamblado y Ejecución Paso a Paso en DOSBox

Hansel Dalí Cerón Rincón - Código: 1152511

### Descripción del Laboratorio

La primera parte del laboratorio se enfoca en la inspección y manipulación básica de la memoria y los registros mediante el uso del depurador, empleando herramientas como el comando R para visualizar el estado inicial del procesador, el comando D para extraer y volcar bloques de memoria verificando patrones específicos, y el comando U para analizar de manera estática los distintos modos de direccionamiento y sus códigos de máquina asociados.  

La segunda parte profundiza en el control de flujo y la gestión de bucles utilizando el registro contador CX, comparando el rendimiento y la eficiencia en bytes de la instrucción nativa LOOP frente a la combinación secuencial de DEC y JNZ. Asimismo, contrasta el uso detallado del comando T para la traza instrucción por instrucción con la automatización del comando G para alcanzar puntos de interrupción en ejecuciones directas.


### Parte 1 - Paso 8: Análisis de la salida del comando D

El comando `D` (Dump) muestra los bytes de la región de memoria recién inicializada organizando la información en tres secciones visuales. 
La primera columna a la izquierda indica las direcciones de memoria donde se encuentran alojados los datos (por ejemplo, `073F:0200`). 
La sección central despliega los valores de los bytes en formato hexadecimal, lo que permite verificar visualmente el patrón cíclico `AB CD EF` que se ha insertado.
La columna de la derecha está destinada a mostrar la traducción de esos mismos bytes a formato ASCII. En este caso, DEBUG imprime únicamente puntos en esa última columna porque los valores `0xAB`, `0xCD` y `0xEF` no corresponden a caracteres ASCII imprimibles, ya que se encuentran fuera del rango válido de `0x20-0x7E`.


### Parte 1 - Paso 12: Verificación No Destructiva de una Escritura en Memoria

Para revisar que lo que escribimos en el Paso 11 quedó bien guardado en memoria y no alterar el estado del programa, la mejor decisión es usar el comando D (Dump).

No es buena idea invocar E 300 a secas (sin pasarle la lista de bytes). Si lo hacemos, el DEBUG entra en modo interactivo: te muestra el byte actual pero se queda esperando a que teclees algo. Ahí es muy fácil embarrarla y presionar una tecla por error, sobrescribiendo el dato sin querer. Es muy diferente a cuando le pasamos toda la lista de parámetros de una vez como en el Paso 11, porque ahí la escritura se hace de un solo golpe, de forma controlada y sin riesgo de equivocarnos a mitad del proceso.

Por otro lado, el comando F (Fill) tampoco nos sirve para esto. Aunque al igual que D también trabaja con rangos de memoria, su funcionamiento es destructivo. Su único propósito es rellenar o "pisar" un bloque de direcciones con un patrón de bytes, así que si lo usamos, borraríamos de una vez los mismos datos que apenas estamos intentando verificar.

Al final, el comando D es el único de los cuatro que de verdad es estrictamente de solo lectura. Mientras que E y F están hechos para escribir y modificar, y el comando R (Registers) te deja alterar los valores internos del procesador si lo llamas con un registro específico (como R AX), el comando D es inmutable. Su única función es leer los bytes de la memoria y mostrarlos en la pantalla en formato hexadecimal y ASCII. Esto es justo lo que necesitamos para auditar el resultado del paso anterior, ya que nos deja ver el estado de la memoria sin meterle ruido al programa ni arriesgar la integridad de los datos que ya cargamos.

### Parte 1 - Paso 14: Modo de Direccionamiento Inmediato vs. Directo a Memoria

Al comparar estas dos instrucciones, la decisión técnica sobre qué modo de direccionamiento usar recae en entender cómo la CPU interactúa con el bus de datos y qué flexibilidad necesitamos en nuestro código.

A pesar de que ambas instrucciones ocupan exactamente 3 bytes en nuestro segmento, el direccionamiento directo a memoria (MOV AX,[0300]) es el que nos va a cobrar un ciclo adicional de acceso al bus de memoria durante su ejecución. La razón es clara si miramos qué contiene el operando fuente: en el direccionamiento inmediato (MOV AX,0005), el valor 0005 ya viene "quemado" o embebido en los bytes de la propia instrucción (B8 05 00). Cuando la CPU extrae la instrucción, ya tiene el dato en la mano. Por el contrario, en el direccionamiento directo (A1 00 03), los bytes solo le indican a la CPU la dirección (0300). El procesador debe decodificar la instrucción y luego realizar un segundo viaje (acceso) a la memoria para resolver esa dirección, leer el dato que está guardado allí y finalmente cargarlo en el registro AX.

A pesar de que este ciclo extra lo hace un poco más lento, como estudiante preferiría el direccionamiento directo a memoria en cualquier escenario donde mi objetivo sea leer un valor que puede cambiar en tiempo de ejecución. Si estamos procesando variables dinámicas —como un dato introducido por el usuario o un valor que nosotros mismos modificamos previamente usando el comando E en el Paso 11—, necesitamos que el programa vaya a buscar el dato actualizado en ese momento. El direccionamiento inmediato no nos sirve aquí, porque nos obligaría a trabajar con una constante fija, estática y ya conocida desde el momento en que ensamblamos el programa.

Para confirmar cómo el comando A codificó cada instrucción sin necesidad de ejecutar el programa (lo cual alterarían los registros), la mejor herramienta es el comando U (Unassemble). Al invocar el comando U pasándole las direcciones donde escribimos nuestro código, el depurador lee el código de máquina directamente de la memoria y lo traduce de vuelta a lenguaje ensamblador en la pantalla. Así, podemos verificar de forma totalmente estática y segura que la primera instrucción generó los bytes B8 05 00 y la segunda A1 00 03, demostrando exactamente qué codificación produjo el ensamblador para cada modo de direccionamiento.


### Parte 2 - Paso 4: Tabla de Traza del Programa de Suma

| Instrucción Ejecutada | Registro AX | Registro BX | Registro CX | Estado de las Banderas |
| :--- | :--- | :--- | :--- | :--- |
| (Estado inicial previo) | `000A` | `0000` | `0000` | `NV UP EI PL NZ NA PO NC` |
| `MOV BX,0005` | `000A` | `0005` | `0000` | `NV UP EI PL NZ NA PO NC` |
| `MOV CX,0003` | `000A` | `0005` | `0003` | `NV UP EI PL NZ NA PO NC` |
| `ADD AX,BX` | `000F` | `0005` | `0003` | `NV UP EI PL NZ NA PE NC` |
| `ADD AX,CX` | `0012` | `0005` | `0003` | `NV UP EI PL NZ AC PE NC` |



### Parte 2 - Paso 7: Traza de Ejecución del Bucle

![Traza del LOOP]

| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿LOOP salta? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Inicial | `MOV CX,0004` | `0000`[cite: 7] | `0004`[cite: 7] | `0106`[cite: 7] | N/A |
| 1 | `ADD AX,0002` | `0002`[cite: 7] | `0004`[cite: 7] | `0109`[cite: 7] | N/A |
| 1 | `LOOP 0106` | `0002`[cite: 7] | `0003`[cite: 7] | `0106`[cite: 7] | Sí |
| 2 | `ADD AX,0002` | `0004`[cite: 7, 8] | `0003`[cite: 7, 8] | `0109`[cite: 7, 8] | N/A |
| 2 | `LOOP 0106` | `0004`[cite: 8] | `0002`[cite: 8] | `0106`[cite: 8] | Sí |
| 3 | `ADD AX,0002` | `0006`[cite: 8] | `0002`[cite: 8] | `0109`[cite: 8] | N/A |
| 3 | `LOOP 0106` | `0006`[cite: 8] | `0001`[cite: 8] | `0106`[cite: 8] | Sí |
| 4 | `ADD AX,0002` | `0008`[cite: 8] | `0001`[cite: 8] | `0109`[cite: 8] | N/A |
| 4 | `LOOP 0106` | `0008`[cite: 8] | `0000`[cite: 8] | `010B`[cite: 8] | No |

**Verificación:** Se confirma que el registro AX alcanza el valor `0x0008` (8 en decimal) exactamente cuando la instrucción `LOOP` finalmente no salta y el Puntero de Instrucción avanza a `010B`[cite: 8]. Esta salida del bucle se produce de forma correcta porque el registro contador CX se reduce a `0x0000`[cite: 8].

### Parte 2 - Paso 10: Selección de Mecanismo de Control de Bucle

Para este punto de la práctica, si solo necesitamos un contador simple y no vamos a usar el registro CX para nada más dentro del ciclo, la opción más lógica es irnos por la instrucción LOOP. Si comparamos el código de máquina, LOOP ocupa apenas 2 bytes (E2 FB), mientras que armar lo mismo separando DEC CX y JNZ nos cuesta 3 bytes (49 75 FA). Al final, esto se traduce en eficiencia pura: con LOOP, el procesador solo tiene que hacer un fetch (extracción de instrucción) por iteración para actualizar el contador y saltar, en lugar de gastar ciclos leyendo dos instrucciones distintas.

Claro que separar el proceso con DEC y JNZ también tiene su utilidad. Yo elegiría esa combinación, a pesar de que ocupe más espacio en memoria, si el programa me obligara a reutilizar el registro CX para hacer otras operaciones matemáticas dentro del ciclo. También sería indispensable si la condición para terminar el bucle no fuera un simple "cuando el contador llegue a cero", sino que dependiera de evaluar un dato específico usando una instrucción de comparación como CMP.

Para comprobar el tamaño de cada alternativa sin tener que correr el programa ni arriesgarnos a alterar el estado de los registros, lo más práctico es usar el comando U (Unassemble). Al desensamblar el segmento de memoria, el DEBUG nos muestra en pantalla el código traducido. Ahí podemos contar visualmente los pares hexadecimales al lado de cada instrucción y confirmar de forma estática cuál de las dos versiones genera menos bytes.


### Parte 2 - Paso 11: Traza del bucle DEC/JNZ con T


| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿JNZ salta? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Inicial | `MOV CX,0004` | `0000` | `0004` | `0206` | N/A |
| 1 | `ADD AX,0002` | `0002` | `0004` | `0209` | N/A |
| 1 | `DEC CX` | `0002` | `0003` | `020A` | N/A |
| 1 | `JNZ 0206` | `0002` | `0003` | `0206` | Sí |
| 2 | `ADD AX,0002` | `0004` | `0003` | `0209` | N/A |
| 2 | `DEC CX` | `0004` | `0002` | `020A` | N/A |
| 2 | `JNZ 0206` | `0004` | `0002` | `0206` | Sí |
| 3 | `ADD AX,0002` | `0006` | `0002` | `0209` | N/A |
| 3 | `DEC CX` | `0006` | `0001` | `020A` | N/A |
| 3 | `JNZ 0206` | `0006` | `0001` | `0206` | Sí |
| 4 | `ADD AX,0002` | `0008` | `0001` | `0209` | N/A |
| 4 | `DEC CX` | `0008` | `0000` | `020A` | N/A |
| 4 | `JNZ 0206` | `0008` | `0000` | `020C` | No |
| Final | `INT 20` | `0008` | `0000` | Terminación | N/A |


### Parte 2 - Paso 12: Comando de Verificación para Bucles de Muchas Iteraciones (T vs. G)

Para enfrentarnos a un bucle de 100 iteraciones (con CX = 0x0064), usar el comando G 20C es muchísimo más práctico porque me ahorra la tediosa tarea de invocar T cientos de veces seguidas. El comando G (Go) deja que el procesador ejecute el código a toda velocidad y se detenga de golpe al alcanzar la dirección que le indico como punto de interrupción. Sin embargo, el costo de esta automatización es que pierdo por completo la visibilidad del proceso interno: dejo de ver cómo evolucionan los registros en cada vuelta y en qué momento exacto cambian las banderas, una información vital si estuviera intentando cazar un bug o un error de lógica a mitad del ciclo.

En el contexto de este laboratorio, fue estrictamente necesario utilizar T en lugar de G durante el Paso 11, cuando armamos la traza de ejecución del bucle. La naturaleza de esa tarea exige documentar el comportamiento del programa de forma microscópica. Para llenar la tabla, necesitaba registrar el estado de AX, cómo disminuía CX y hacia dónde apuntaba el IP iteración por iteración. El comando G solo me habría arrojado la foto final del programa, pero armar una tabla de traza requiere exactamente esa granularidad de avanzar instrucción por instrucción que solo el comando T puede ofrecer.

Finalmente, después de ejecutar G 20C y plantarme en la instrucción final, si quisiera confirmar únicamente el valor con el que quedó el registro AX sin tener que observar ningún paso intermedio ni correr más código, el comando que usaría sería R AX (o simplemente R para proyectar el estado estático de todos los registros en ese preciso momento).