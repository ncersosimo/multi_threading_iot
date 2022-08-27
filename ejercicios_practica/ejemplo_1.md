# Ejercicios de práctica

En esta práctica no es necesario utilizar ningún simulador, trabajaremos sobre el los ejemplos construidos de la aplicación completado "bridge".

Logearse desde VM y obtener cual es la dirección IP del dispositivo:
```sh
$ ifconfig
```

### 1 - Objetivo
Dentro de la carpeta "ejercicios_practica" cree un script llamado "ejercicio_1"

El objetivo es sumar más funcionalidades al sistema desde el lado de monitoreo de salud del dispositivo. En este caso usted puede realizar este desafio tomando como punto de partida el ejemplo de clase completo de punete (bridge), a lo que agregará:
- Dentro de la función on_connect_remoto debe agregar la posibilidad de escuchar por el tópico "keepalive/request", recuerde que el tópico debe tener antes la parte base según el usuario del campus en la variable de config:
```
dashboardiot/<usuario_campus>/keepalive/request
```
- Dentro de la función de procesar remoto deberá analizar si recibe el tópico "keepalive/request", recuerde que antes del tópico está el tópico base que debe descartar:
```
dashboardiot/<usuario_campus>/keepalive/request
```
- Este tópico es enviado por el dashboard una vez por segundo luego de que usted se logea en el con su usuario del campus.
- Debe tener configurado en el archivo .env el mismo usuario de campus que utiliza para logearse para que le llege este mensaje.
- En caso de recibir este mensaje usted debe enviar una "1" al siguiente tópico:
```
dashboardiot/<usuario_campus>/keepalive/ack
```

### 2 - Verificación
Si todo funciona correctamente, en la sección de usuarios del dashboard deberá ver como aparece su usuario y se actualiza la cantidad de paquetes recibidos una vez por segundo.

En caso de que otros alumnos estén haciendo el desafio al mismo tiempo también recibirá en el dashboard sus mensajes con sus usuarios del campus.

Utilice todas las herramientas a su disposición (terminal, MQTTExplorer, debugger) para ensayar y testear el funcionamiento de su implementación. En caso que tenga problemas, consulte y continue explorando. Lo más rico de estos ejercicios es que pueda analizar las fallas y aprender de ellas por su cuenta como todo un buen detective.

Una vez finalizado el ejercicio y corroborado el funcionamiento, subir al repositorio el script de python resuelto de este ejercicio en la carpeta de "ejercicios_practica" con el nombre de "ejercicio_1.py".
