## Ejemplos de clase

### 1 - Preparar el entorno de trabajo

Logearse desde VM y obtener cual es la dirección IP del dispositivo:
```sh
$ ifconfig
```

Abrir el Visual Studio Code y conectarse de forma remota al dispositivo. Trabajaremos sobre la carpeta recientemente clonada del repositorio.

### 1 - Vistazo desde donde comenzamos
Desde el VSC abrir el script de "ejemplo_5_bridge.py", el cual viene con todo lo realizado en el ejemplo de ETL. El objetivo será crear un thread para el procesamiento de datos provenientes del MQTT remoto (dashboard), por lo que observe lo siguiente:
- Observe que el cliente_remoto no se suscribe a ningún tópico del dashboard en su función de "on_connect_remoto". Deberá agregar los tópicos que desea eschar según el usuario definido en el archivo ".env".
- Entre los callbacks de cliente_remoto verá que hay un comentario que hace referencia "procesamiento_remoto", el cual implementaremos en este ejemplo.
- Deberemos crear un nuevo thread para manipular lo recibido del cliente_remoto.
- Debemos esperar al final del programa a que los threads terminen para terminar el programa (join), incluyendo este nuevo thread.


### 2 - Tópicos MQTT remoto
LO primero será suscribirnos en la función "on_connect_remoto" a los tópicos que nos interesan escuchar desde el dashboard:
- Estos tópicos serán de aquellos actuadores que pueden ser controlados desde el dashboard (luces, motores).
- Deberemos suscribirnos unicamente a aquellos tópicos enviados a nuestro usuario, definido en el archivo ".env".

Agregar el siguiente bloque de código dentro de la función "on_connect_remoto" que nos permita conectarnos a los actuadores del dashboard:
```python
# Aquí Suscribirse a los topicos remotos deseados
client.subscribe(config["DASHBOARD_TOPICO_BASE"] + "actuadores/volar")
client.subscribe(config["DASHBOARD_TOPICO_BASE"] + "actuadores/luces/1")
client.subscribe(config["DASHBOARD_TOPICO_BASE"] + "actuadores/motores/#")
```

A continuación, dentro de la función "on_message_remoto" colocaremos un print que nos permita verificar que el programa está recibiendo los tópicos de nuestro usuario:
```python
topico = message.topic
mensaje = str(message.payload.decode("utf-8"))
print("DASHBOARD:", topico, mensaje)
```

### 3 - Verificación
- Lance el programa y verifique en la consola que cada cliente (local y remoto) se hayan conectado.
- Ingrese al dashboard de con su usario del campus, manipule algunos de los actuadores.
- Verifique en la consola que los mensajes del DASHBOARD llegaron correctamente a la función de "on_message_remoto".


### 4 - Crear thread de procesamiento_remoto
Primero de todo debemos dentro de la función "on_mesesage_remoto" enviar el mensaje a la queue para que puede ser luego procesado:
```python
topico = message.topic
mensaje = str(message.payload.decode("utf-8"))
print("DASHBOARD:", topico, mensaje)
queue.put({"topico": topico, "mensaje": mensaje})
```

Entre los callbacks definidos para el cliente_remoto, definir esta función que utilizaremos como callback de procesamiento del cliente_remoto:
```python
# Aquí crear el callback procesamiento_remoto
def procesamiento_remoto(name, flags, client_local, client_remoto):
    print("Comienza thread", name)
    queue = client_remoto._userdata["queue"]

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
        
        # Quitar la parte del tópico que corresponde al dashboard y el usuario
        topico_local = topico.replace(config["DASHBOARD_TOPICO_BASE"], '')
        # Agregar el destintivo de que el mensaje viene del dashboard
        topico_local = "dashboardiot/" + topico_local
        # Enviar el mensaje al cliente MQTT local para que otros
        # acutadores o sensores estén al tanto de lo recibido
        client_local.publish(topico_local, mensaje)

    print("Termina thread", name)
```

Observe que la función que cumple procesamiento_remoto:
- Tomar los mensajes que llegan del dashboard y los envía al cliente MQTT local.
- Para eso la función quitar del tópico el string relativo al dashboard y al usuario en sí ya que no son datos necesarios dentro de la red local de MQTT.
- Además antepone al tópico "dashboardiot/" para evitar el efecto loopback (diferenciar de donde proveien el mensaje recibido). Los simuladores "drone_iot" y "drone_mock_iot" están preparados para soportar estos tópicos.

La función antes definida la lanzaremos con un thread. Crearemos este thread justo después de la definición del thread de procesamiento local:
```python
print("Lanzar thread de procesamiento de MQTT remoto")
thread_procesamiento_remoto = threading.Thread(target=procesamiento_remoto, args=("procesamiento_remoto", flags, client_local, client_remoto), daemon=True)
thread_procesamiento_remoto.start()
```

En la definición del thread estaremos enviado a la función:
- Los flags para finalizarlo de forma segura.
- El cliente local y remoto para enviar o recibir mensajes.

Lo último que nos queda es esperar la correcta finalización del thread luego del bucle principal del programa. Agregar el "join" del thread de procesamiento_remoto justo luego del "join" de procesamiento_local:
```python
thread_procesamiento_local.join()
thread_procesamiento_remoto.join()
```

### 5 - Conectar con el dashboard
Verificar los mensajes recibidos del dashboard de actuadores (luces o motores) tiene impacto en el compartamiento de los simuladores. Puede utilizar alguno de los simulares que se sugieren en el README para validar que los sensores o actuadores modificados en el simulador se ven reflejados en el dashboard.