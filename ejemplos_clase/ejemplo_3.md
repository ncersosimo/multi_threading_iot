## Ejemplos de clase

### 1 - Preparar el entorno de trabajo

Logearse desde VM y obtener cual es la dirección IP del dispositivo:
```sh
$ ifconfig
```

Abrir el Visual Studio Code y conectarse de forma remota al dispositivo. Trabajaremos sobre la carpeta recientemente clonada del repositorio.

### 1 - Vistazo desde donde comenzamos
Desde el VSC abrir el script de "ejemplo_3_sigint.py", el cual viene con todo un sistema de threads de "productor" "consumidor" de mensajes enviados por consola:
- Observe como la variable queue es creada y compartida entre los threads pasada como parámetro al momento de crearlos.
- Observe que el thread "consumidor" quedará bloqueado hasta recibir algo en la queue.
- Observe queel thread "productor" quedará bloqueado hasta recibir un texto por consola.
- Observe que el bucle thread del programa princiapal (While True) quedará bloqueado en ese bucle por siempre.


### 2 - Lanzar el programa
Al lanzar el programa verá que le thread "productor" le solicitará por consola que ingrese un texto. Lance el programa y vea como funciona:
```sh
$ python3 ejemplo_3_sigint.py
```

Podrá ver que el programa nunca finaliza, la unica forma de cortar su ejecución es con las teclas "CTRL + C" en la consola o utilizando el comando:
```sh
$ ps -a
$ kill <PID>
```


### 3 - Función finalizar programa
A fin de finalizar el programa correctamente, capturaremos la señal que es enviada al programa cuando se presionan las teclas "CTRL + C" en el teclado. Esto envia una señal de SIGINT.
- Observe que en el programa hemos creado una variable de "flags" con el siguiente contenido:
```python
flags = {"thread_continue": True}
```

Utilizaremos esa variable para indicarle a los threads que llegó la hora de finalizar. Debemos por lo tanto agregar una función que capture la señal SIGINT y modifique la variable de "thread_continue".

Debajo de la definicion de los flags de thread, antes del bloque princial, definir la función de cierre como:
```python
flags = {"thread_continue": True}
def finalizar_programa(sig, frame):
    global flags
    print("Señal de terminar programa")    
    flags["thread_continue"] = False
```

Luego del bloque de código relativo a la creación de todos los threads, agregue la siguiente linea de código en el bloque principal del programa para conectar el cierre de la aplicación con la función "finalizar_programa":
```python
# Capturar el finalizar programa forzado
signal.signal(signal.SIGINT, finalizar_programa)
```

Lance el programa y analice ahora su comportamiento al presionar "CTRL + C" en la consola luego de lanzarlo:
```sh
$ python3 ejemplo_3_sigint.py
```

Podrá observar que consola que el programa captura la petición de cierre con "CTRL + C" pero este no se detiene. Nos falta agregar la lógica de finalizar programa, por ahora podremos detener el thread con:
```sh
$ ps -a
$ kill <PID>
```

### 4 - Finalizar programa correctamente
Los threads que acabamos de generar necesitarán esta variable de "flags" para saber cuando finaliza un thread, para ello debemos pasar la variable como parámetro al momento de su creación:
```python
args=("producir_datos", queue, flags)
...
args=("consumir_datos", queue, flags)
```

También debemos modificar la definición de las funciones "producir_datos" y "consumir_datos":
```python
def producir_datos(name, queue, flags):
...
def consumir_datos(name, queue, flags):
```

Los threads ya están listos para usar la variable "flags". Ahora debemos reemplazar los bucles infinitos (While True) con el siguiente código:
```python
while flags["thread_continue"]:
```

Esta acción se debe repetir para todos los bucles infinitos de cada thread:
- El thread producir_datos
- El thread consumidor_datos
- El thread principal


### 5 - Verificación
Lanzar el programa e intentar detenerlo con las teclas "CTRL + C":
- Verá en la consola el print "Señal de terminar programa" indicando que se está finalizando el programa.
- Para que los threads se despierten y finalicen debe enviar un texto por consola.
- Al enviar el texto por consola se despierta el thread "producir_datos" y termina.
- La acción anterior envia un mensaje por la queue despertando al thread "consumir_datos" y termina.

En próximos ejemplos, debemos enviar un mensaje nulo "queue.put(None)" antes de los "join" para que todos los threads se despierten y finalicen también.

