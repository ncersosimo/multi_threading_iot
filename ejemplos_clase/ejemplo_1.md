## Ejemplos de clase

### 1 - Preparar el entorno de trabajo

Logearse desde VM y obtener cual es la dirección IP del dispositivo:
```sh
$ ifconfig
```

Conectarse por ssh desde una terminal del host:
```
$ ssh inove@<ip_dispositivo>
```

Para descargar el repositorio de esta clase, crear la carpeta "clase_4" para trabajar sobre los ejemplos de esta clase:
```sh
$ mkdir clase_4
```

Ingresar a la carpeta creada y clonar la carpeta del repositorio de esta clase:
```sh
$ cd clase_4
$ git clone https://github.com/InoveAlumnos/multi_threading_iot
$ cd multi_threading_iot
```

Abrir el Visual Studio Code y conectarse de forma remota al dispositivo. Trabajaremos sobre la carpeta recientemente clonada del repositorio.

### 1 - Vistazo desde donde comenzamos (repaso clase anterior)
Desde el VSC abrir el script de "ejemplo_1_queue.py", el cual viene con la conectividad a MQTT resuelta (local y remota).
- Observe como se ha creado un cliente que utilizaremos para el MQTT local (cliente_local) y MQTT remoto (client_remoto)
- Observa como se han creado las funciones de "on_connect"  y "on_message" tanto para el cliente local como el cliente remoto.

### 2 - Crear una queue para MQTT 
Dentro del bloque de programa principal, justo antes de crear el cliente para el MQTT local crear una queue:
```python
# crear una queue para el cliente local --> queue_local
queue_local = Queue()
```

Siguiente paso es dejar disponible esta variable "queue_local" dentro del objeto del cliente_local para que pueda ser utilizada en "on_message". Para eso, justo antes de conectar el MQTT local dejar disponible la variable queue en user_data:
```python
# Dejar disponible la queue dentro de "user_data"
client_local.user_data_set( 
    {
        "queue": queue_local,
    }
)
```

Dentro de la función "on_message_local" agregar a la queue (put) cada mensaje nuevo que llegue con. A fin de almacenar el tópico y el mensaje de entrada utilice la siguiente estructura:
```python
# Agregar cada nuevo mensaje a la queue (put)
queue.put({"topico": topico, "mensaje": mensaje})
```

Dentro del bucle principal del programa (While True) espere por nuevos mensajes en la queue (get). Cuando llegue un nuevo mensaje tome el tópico y el mensaje e imprimalos en la consola:
```python
    # El programa principal quedará a la espera
    # de que llegue un nuevo mensaje MQTT
    while True:
        msg = queue_local.get(block=True)
        # Hay datos para leer, los consumo e imprimo en consola
        topico = msg['topico']
        mensaje = msg['mensaje']
        print(f"{msg['topico']}: {msg['mensaje']}")
```

### 3 - Verificación
- Lance el programa y verifique en la consola que cada cliente (local y remoto) se hayan conectado.
- Envie un mensaje a los tópicos que espera el cliente local.
- Verifique en consola que el mensaje haya llegado a la queue y se haya mostrado correctamente en consola.

### 4 - Conectar con el dashboard
Dentro del bucle principal del programa que acaba de modificar (While True), agregue la información necesaria al tópico para que el mensaje llegado a la queue puede llegar al dashboard:
```python
print(f"{msg['topico']}: {msg['mensaje']}")        

topico_remoto = config["DASHBOARD_TOPICO_BASE"] + topico
client_remoto.publish(topico_remoto, mensaje)
```
