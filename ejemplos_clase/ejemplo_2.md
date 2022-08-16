## Ejemplos de clase

### 1 - Preparar el entorno de trabajo

Logearse desde VM y obtener cual es la dirección IP del dispositivo:
```sh
$ ifconfig
```

Abrir el Visual Studio Code y conectarse de forma remota al dispositivo. Trabajaremos sobre la carpeta recientemente clonada del repositorio.

### 1 - Vistazo desde donde comenzamos
Desde el VSC abrir el script de "ejemplo_2_threads.py", el cual contiene dos threads creados. El objetivo será analizar como estos threados funcionan observando la consola y como cerrarlos o terminar el programa adecuadamente:
- ¿Cómo cree que se comportarán los threads? ¿El thread_dos no comenzará hasta haber finalizado el thread_uno?
- Observe que sucede si lanzamos el programa, ¿falla cierto?


### 2 - Finalizar programa correctamente
Agregar al final del programa los "join" de cada thread para esperar a que cierren definitivamente:
```python
# No puedo finalizar el programa sin que hayan terminado los threads
# el "join" espera por la conclusion de cada thread, debo lanzar el join
# por cada uno
print("Espero que termine thread one")
thread1.join()
print("Espero que termine thread two")
thread2.join()
```

Lanzar el programa desde la consola y analizar como funciona:
```sh
$ python3 ejemplo_2_threads.py
```

### 3 - Matar un proceso
Acabamos de ver que el programa no finaliza hasta que todos los threads hayan finalizado. ¿Cómo podemos detener un programa o forzar que termine en caso de que los threads nunca finalicen?

Antes de lanzar el programa nuevamente, observemos que procesos están en ejecución lanzando en UNA NUEVA CONSOLA el siguiente comando:
```sh
$ ps -a
```

Ahora lanzaremos el programa nuevamente en nuestra consola, y luego observaremos en la segunda consola que proceso nuevo se ha lanzado.

Consola uno:
```sh
$ python3 ejemplo_2_threads.py
```

Consola dos:
```sh
$ ps -a
```

Seguramente se encontrará con un nuevo proceso que lleva el nombre de la versión de python que utilizó para lanzar el programa, copio el ID de ese proceso (PID) y matelo con el siguiente comando:
Consola dos:
```sh
$ kill <PID>
```



