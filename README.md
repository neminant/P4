PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/mod/resource/view.php?id=3654387?forcedownload=1)
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuerde que, además de los trabajos indicados en esta parte básica, también deberá realizar un proyecto
de ampliación, del cual deberá subir una memoria explicativa a Atenea y los ficheros correspondientes al
repositorio de la práctica.

A modo de memoria de la parte básica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.
  
    **sox**: Sirve para cambiar el formato de una señal de entrada a uno de salida que nos convenga. Para saber las características de sox, escribimos sox -h en el terminal. Para la conversión se puede elegir cualquier formato de la señal de entrada y los bits utilizados entre otras cosas. 
  
    **$X2X**: Programa de sptk que sirve para transformar datos input a otro formato output. La manera de utilizar este comando en el terminal es la siguiente: x2x [+type1 [+type2][–r] [–o] [%format].
  
   **FRAME**: Extrae el frame de la secuencia de datos. -l indica la longitud del frame, y -p indica el periodo del frame.
  
   **WINDOW**: Enventanado de ventana. -l indica la longitud de frames del input. -L indica la longitud de frames del output.
  
  **LPC**: Calcula los coeficientes de predicción lineal. -l indica la longitud de frames, y -m indica el orden de coeficientes LPC.

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 51 del script `wav2lp.sh`).

  Primero de todo, se obtiene el fichero $base.lp con los coeficientes LPC, encadenando los comandos descritos en el apartado anterior.

  Segundo, se fija una cabecera para el archivo de salida con el número de filas(`nrow`) y columnas de la matriz(`ncol`). El número de columnas será el orden del LPC + 1, puesto que en la primera columna se encuentra el factor de ganancia. El número de filas será el número total de tramas a las que se les ha calculado los coeficientes LPC. Se extrae del fichero .lp convirtiendo el contenido a ASCII con X2X +fa y contando el número de líneas con el comando wc -l.

  * ¿Por qué es más conveniente el formato *fmatrix* que el SPTK?

    Porque se tiene un fácil y rápido acceso a todos los datos almacenados con una correspondencia directa entre la posición en la matriz y el orden del coeficiente y número de trama, por lo que simplifica mucho su manipulación a la hora de trabajar. También nos ofrece información directa en la cabecera sobre el número de tramas y número de coeficientes calculados.

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:

  ```sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 | $LPC -l 240 -m $lpc_order | $LPC2C -m $lpc_order -M $nceps > $base.lpcc```

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:

  ```sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $MFCC -s 8000 -n $nfiltros -l 240 -m $mfcc_order > $base.mfcc```

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
![image](https://github.com/neminant/P4/assets/125289603/59562047-d4d5-4d17-a998-0e853c86b489)


  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
    
fmatrix_show work/lp/BLOCK01/SES010/*.lp | egrep '^[' | cut -f4,5 > lp_2_3.txt

fmatrix_show work/lpcc/BLOCK01/SES010/*.lpcc | egrep '^[' | cut -f4,5 > lpcc_2_3.txt

fmatrix_show work/mfcc/BLOCK01/SES010/*.mfcc | egrep '^[' | cut -f4,5 > mfcc_2_3.txt

  + ¿Cuál de ellas le parece que contiene más información?

Para que una parametrización contenga más información que otra, debe tener los coeficientes más incorrelados entre sí porque no queremos información redundante.

Por tanto, y observando las tres gráficas, podemos deducir que los coeficientes más incorrelados son los que se encuentran más dispersos, es decir, en nuestro caso, los coeficientes MFCC. Aún así, observamos que los coeficientes LPCC también están muy disperso y en consecuencia, incorrelados.

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.
  
  pearson work/lp/BLOCK01/SES010/*.lp
  
  ![image](https://github.com/neminant/P4/assets/125289603/6cb1d15e-b130-4948-a7f8-aa5cd4bc7bb2)
  
  pearson work/lpcc/BLOCK01/SES010/*.lpcc
  
  pearson work/mfcc/BLOCK01/SES010/*.mfcc


  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | &rho;<sub>x</sub>[2,3] |   -0,664123   |      |      |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
 
    Un valor de |ρx[2,3]| alto nos indica que los coeficientes están muy correlados y un valor bajo nos indica que los coeficientes están poco correlados (|ρx[2,3]| ∊ [0,1]). Si nos fijamos en la tabla, observamos que los resultados de pearson concuerdan con las conclusiones obtenidas al analizar las gráficas: LPC es la parametrización que nos aporta menos información con diferencia, MFCC la que más, ya que sus coeficientes están muy poco correlados, y LPCC nos aportá un poco menos de información que MFCC. Esto nos ayuda a ver que MFCC será la parametrización adecuada para optimizar nuestro sistema con diferencia.
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

  Para los coeficientes LPCC se usa lpc_order=8, como esta definido en la función compute_lp(), y el número de cepstrum es igual a 3P/2 donde P=lpc_order=8 , por lo tanto, nceps=12. Finalmente, hemos decidido incrementar estos valores para obtener mejores resultados.

Para los coeficientes MFCC se usan los primeros 13 coefficientes + un 50% más, por lo tanto mfcc_order=19. I el numero de filtros suele ir de 24 a 40, por lo que usamos un valor intermedio de nfilter=30.

### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.

- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.

### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.

### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
 
### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.
