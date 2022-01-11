PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/pluginfile.php/3145524/mod_assign/introattachment/0/spk_8mu.tgz?forcedownload=1)
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

   En el script wav2lp.sh encontramos el siguiente pipeline principal:
  
  ![image](https://user-images.githubusercontent.com/91891304/148940465-2afd6125-f235-4a7f-a8ef-39a1a63afd04.png)
   
   sox: nos genera la señal en formato *raw* convirtiendo el fichero de entrada a un formato de enteros con signo “signed-integer” de 16 bits/muestra a partir de las            siguientes opciones en el fichero de entrada:
	    -t: formato (raw)
	    -e: codificación (signed-integer)
	    -b: bits/muestra en -e (16)

  $X2X: Permite la conversión entre distintos formatos de datos, que en nuestro caso la convierte a float (+sf).

  $FRAME: Extrae o divide una o más tramas de una secuencia de datos. En nuestro caso, con -l indicamos la longitud (240) del segmento en la que divide las muestras, y con -p   el periodo (80) de desplazamiento entre ellas.

  $WINDOW: Le indicamos el tamaño de la ventana tanto en su input (240) con -l, como en su output (240) con -L.

  $LPC: Nos calcula los coeficientes de la predicción lineal (LPC). Le indicamos la longitud de trama (240) con -l, el orden de LPC con -m (con la variable $lpc_order), y el   fichero output.
  

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 47 del script `wav2lp.sh`).
  
  ![image](https://user-images.githubusercontent.com/91891304/148940752-0db74354-f6ce-4760-b045-590f7f8ac505.png)

  Para la matriz fmatrix necesitamos el número de filas ($nrow) y columnas ($ncol). 
  
  $ncol: Tiene que ser igual al número de coeficientes, por lo que lo obtenemos simplemente de sumarndo ‘1’ al orden del LPC ($lpc_order+1).
  
  $nrow: Primero convertimos el contenido de $base.lp de float (+f) a ASCII (+a) con $X2X, y después contamos el número de líneas del fichero ASCII y para obtener el número     de filas de fmatrix con wc -l. Para imprimir la matriz separando filas y columnas con un salto de línea usamos perl -ne.

  
  * ¿Por qué es conveniente usar este formato (u otro parecido)? Tenga en cuenta cuál es el formato de
    entrada y cuál es el de resultado.
    
    *fmatrix* nos permite pasarle un fichero de datos (en nuestro caso “base.lp”) y nos lo "ordena" como float en “nrow” filas y “ncol” columnas.
    
    De esta forma, con “fmatrix_show” podremos ver los datos de forma sencilla, y situarnos en la posición de la matriz que nos interesa sabiendo el número del audio para         encontrar los coeficientes de este.
    

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:
  
  Usando el “'wav2lp.sh'” como plantilla, solo tenemos que cambiar los inputs de entrada, y las variables asignadas en el “sox”:
  
  ![image](https://user-images.githubusercontent.com/91891304/148941056-f85ef1bb-70e2-4cb3-909f-8e1de53dac8c.png)

  
- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:
  
  ![image](https://user-images.githubusercontent.com/91891304/148941197-39aed0fc-435a-4858-8ca2-de5fa83cc517.png)


### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  
  ![image](https://user-images.githubusercontent.com/91891304/148941235-28957ae5-7158-474e-b90e-9724da2ce1ff.png)
  
  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
    
    Parametrizaciones LP, LPCC, MFCC:
    fmatrix_show work/lp/BLOCK01/SES017/*.lp | egrep '^\[' | cut -f4,5 > lp_2_3.txt
    fmatrix_show work/lpcc/BLOCK01/SES017/*.lpcc | egrep '^\[' | cut -f4,5 > lpcc_2_3.txt
    fmatrix_show work/mfcc/BLOCK01/SES017/*.mfcc | egrep '^\[' | cut -f4,5 > mfcc_2_3.txt
    
    Y luego con el código “graph1.py” en Python obtenemos la gráfica.
    
  + ¿Cuál de ellas le parece que contiene más información?

    Contra más incorrelado sea, mayor será la información de un coeficiente.
    
    Sabiendo esto, como en la gráfica superior vemos como la distribución de los coeficientes de MFCC está mucho más desperdigada que la de LP y LPCC, podemos decir que la       incorrelación es más grande y la dependencia entre coeficientes más pequeña, y por tanto hay más información en MFCC que en las otras parametrizaciones.


- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.
  
  Con los comandos siguientes, generamos para el locutor BLOCK01/SES017 un fichero de texto para cada parametrización.
  
  pearson -N work/lp/BLOCK01/SES017/*.lp >lp_pearson.txt
  pearson -N work/lpcc/BLOCK01/SES017/*.lpcc >lpcc_pearson.txt
  pearson -N work/mfcc/BLOCK01/SES017/*.mfcc >mfcc_pearson.txt

  |                        |     LP    |   LPCC   |    MFCC   |
  |------------------------|:---------:|:--------:|:---------:|
  | &rho;<sub>x</sub>[2,3] | -0.874552 | 0.151453 | -0.105998 |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
    
    Como podemos ver, hay más relación entre coeficientes en la correlación normalizada de MFCC que con los de LP y LPCC, ya que este es el que tiende más a cero.
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

  Para el LPCC: Un orden LPC de 14 (valor cercano a 13) y un orden del cepstrum de 21.
  
  Para MFCC: 14 coeficientes (suele ser 13) y 40 filtros (suelen ser entre 24 y 40).


### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
  Con el siguiente comando vamos a generar un fichero SES009.gmm:
  plot_gmm_feat work/gmm/mfcc/SES009.gmm
  
  ![image](https://user-images.githubusercontent.com/91891304/148941722-ed5ef232-fbab-439c-b4da-ea1a520d1337.png)

  Para que quede mejor, primero entrenamos la SES y después tiramos el comando inferior para crear la gráfica pedida:

  gmm_train -d work/lp -e lp -g SES009.gmm -m 10 -N 100000 -T  0.0001 -i 2 lists/class/SES009.train
  python3 scripts/plot_gmm_feat.py SES009.gmm

  ![image](https://user-images.githubusercontent.com/91891304/148941766-6e8150ab-8488-4e3d-9849-6d662d984293.png)

  - Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.
  
  Para mostrar la gráfica del SES009 con las muestras del locutor correspondiente:
  plot_gmm_feat work/gmm/mfcc/SES009.gmm work/mfcc/BLOCK00/SES009/SA009S* &
  
  ![image](https://user-images.githubusercontent.com/91891304/148942116-4374bc9c-e211-43a1-be32-27517c0a3833.png)
  
  Con esto, ya podemos usar las muestras de otro locutor y comprobar si la modulación es mejor o peor según si la correlación entre las muestras y la región estimada es mayor   o menor. En este caso elegimos SES008 para hacerlo:
  
  plot_gmm_feat work/gmm/mfcc/SES017.gmm work/mfcc/BLOCK01/SES016/SA016S* &

  ![image](https://user-images.githubusercontent.com/91891304/148942163-afadb2bf-0425-42fb-a83c-13e2b96ebfea.png)


### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.
  
  | Parámetros |  Errores  |   Total  |  Tasa de error |
  |------------|:---------:|:--------:|:--------------:|
  | LP         | 79        | 785      | 10.06%         |
  |------------|:---------:|:--------:|:--------------:|
  | LPCC       | 152       | 785      | 19.36%         |
  |------------|:---------:|:--------:|:--------------:|
  | MFCC       | 125       | 785      | 15.92%         |


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
