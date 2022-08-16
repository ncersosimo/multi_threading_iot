## Ejemplos de clase

### 1 - Preparar el entorno de trabajo

Logearse desde VM y obtener cual es la dirección IP del dispositivo:
```sh
$ ifconfig
```

Abrir el Visual Studio Code y conectarse de forma remota al dispositivo. Trabajaremos sobre la carpeta recientemente clonada del repositorio.

### 1 - Vistazo desde donde comenzamos
Desde el VSC abrir el script de "ejemplo_4_etl.py", el cual viene con todo lo realizado en el primero ejemplo de queue con MQTT. El objetivo será crear un thread para el procesamiento de datos fuera del bucle princial, por lo que observe lo siguiente:
- Observe que NO contamos con un mecanismo seguro para cerrar los threads, debemos utilizar la variable flags["thread_continue"] para indicarles a todos los threads que deberán cerrarse.
- Entre los callbacks de cliente_local verá que hay un comentario que hace referencia "procesamiento_local", el cual implementaremos en este ejemplo.
- Debemos esperar al final del programa a que los threads terminen para terminar el programa (join).


### 2 - Función finalizar programa
Debajo de la definicion de los flags de thread, antes del bloque princial, definir la función de cierre como:
```python
def finalizar_programa(sig, frame):
    global flags
    print("Señal de terminar programa")    
    flags["thread_continue"] = False
```

Luego del bloque de código relativo a la conexión de todos los clientes (remoto y local), agregue la siguiente linea de código en el bloque principal del programa para conectar el cierre de la aplicación con la función "finalizar_programa":
```python
# Capturar el finalizar programa forzado
signal.signal(signal.SIGINT, finalizar_programa)
```

Modifque el bucle principal del programa (While True) para que se quede atento a esta señal y variable:
```python
# El programa principal queda a la espera de que se desee
# finalizar el programa
while flags["thread_continue"]:
    # busy loop
    time.sleep(0.5)
```

### 3 - Verificación
- Lance el programa y verifique en la consola que cada cliente (local y remoto) se hayan conectado.
- Verifique que al cerrar el programa con "CTRL+C" este se cierre correctamente


### 4 - Crear thread de procesamiento_local
Entre los callbacks definidos para el cliente_local, definir esta función que utilizaremos como callback de procesamiento del cliente_local:
```python
# Aquí crear el callback procesamiento_local
def procesamiento_local(name, flags, client_local, client_remoto):
    print("Comienza thread", name)
    queue = client_local._userdata["queue"]

    while flags["thread_continue"]:
        # Queue de python ya resuelve automaticamente el concepto
        # de consumidor con "get".
        # En este caso el sistema esperará (block=True) hasta que haya
        # al menos un item disponible para leer        
        msg = queue.get(block=True)

        # Sino hay nada por leer, vuelvo a esperar por otro mensaje
        if msg is None:
            continue

        # Hay datos para leer, los consumo e imprimo en consola
        print(f"mensaje recibido en thread {name}:")
        print(f"{msg['topico']}: {msg['mensaje']}")
        topico = msg['topico']
        mensaje = msg['mensaje']
        
        topico_remoto = config["DASHBOARD_TOPICO_BASE"] + topico
        client_remoto.publish(topico_remoto, mensaje)

    print("Termina thread", name)
```

La función antes definida la lanzaremos con un thread. Crearemos este thread justo antes de comenzar el bucle principal:
```python
# El programa principal solo armará e invocará threads
print("Lanzar thread de procesamiento de MQTT local")
thread_procesamiento_local = threading.Thread(target=procesamiento_local, args=("procesamiento_local", flags, client_local, client_remoto), daemon=True)
thread_procesamiento_local.start()
```

En la definición del thread estaremos enviado a la función:
- Los flags para finalizarlo de forma segura.
- El cliente local y remoto para enviar o recibir mensajes.

Lo último que nos queda es esperar la correcta finalización del thread luego del bucle principal del programa. Para eso debemos agregar las siguientes líneas para enviar un mensaje a la queue para despertar al thread y que termine, y realizar el "join" a la espera de que realmente termine el thread:
```python
# Se desea terminar el programa, desbloqueamos el thread
# con un mensaje vacio y se lo espera con un join
queue_local.put(None)
thread_procesamiento_local.join()
```

### 5 - Conectar con el dashboard
Verificar que en el dashbard llegan los mensajes esperados desde el MQTT local. Puede utilizar alguno de los simulares que se sugieren en el README para validar que los sensores o actuadores modificados en el simulador se ven reflejados en el dashboard.