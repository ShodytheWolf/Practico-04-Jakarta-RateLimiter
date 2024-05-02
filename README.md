22-04-2024 | 14:16
Status: #Process 
Tags: [JavaEE](../JavaEE.md)

---

Hoy estáremos realizando el [práctico N° 4 de JavaEE](https://docs.google.com/document/d/1iSOkuzRDW268ouRyYSDoaxjdCSaPSjXJCyKo7qUpT44/edit#heading=h.gugr02j824rw) 

En donde estáremos viendo [Rate limiter](../JavaEE/18-04-24/Rate%20limiter.md) y validación de credenciales utilizando [Identity Store](../JavaEE/18-04-24/Identity%20Store.md)

# Parte A
### Ejercicio 1
*Descargue JMeter del sitio oficial, descomprima la carpeta y ejecute la [aplicación](https://jmeter.apache.org/download_jmeter.cgi)
nota: la alternativa de instalarla en el sistema sudo apt install jmeter (no instala la última versión)*

*Instale los plugins utilizados para ejecutar los ejemplos que vimos en clases:*
*siga las instrucciones para descargar el [plugin manager](https://jmeter-plugins.org/install/Install/)*
*en el menú de JMeter->opciones->Plugin Manager. seleccione los plugins BlazeMeter*

Se descarga *JMeter 5.6.3* desde la página como indica la consigna, y siguiendo los pasos presentados en la documentación del [Plugin Manager](https://jmeter-plugins.org/install/Install/), se instalan los plugins necesarios para poder observer el correcto funcionamiento de del testeo de nuestro servidor.

Aquí se muestran los plugins instalados que se utilizarán durante el resto de este documento:
![](_attachments/Pasted%20image%2020240423133254.png)

Observese como, además de *BlazeMeter* se han instalado otros plugins, tales como *5 Additional Graphs* para poder gráficar más tipos de datos, o *jpgc - Standard Set*
que añade más funcionalidades a los testeos que pronto haremos.

---
# Parte B
### Ejercicio 1
*"Descargue el proyecto 03b_JakartaEESecuriry."*

Se descarga el proyecto de Jakarta Security que nos proporcionó el profesor en el siguiente repositorio: https://github.com/gabrielaramburu/TallerJakartaEE

---
### Ejercicio 2
*"Observe las particularidades del pom.xml"*

Tras examinar el POM.xml del proyecto, me encuentro con que hay que configurar un par de cosas antes de poder proseguir:
![](_attachments/Pasted%20image%2020240423135348.png)

Tal y como dice el comentario dejado por el profe, debido a que el servidor provisto con
```Bash
mvn package wildfly:dev
```
No cumple con la especificación de [Security de Jakarta 3](https://jakarta.ee/specifications/security/3.0/)

debemos conectarnos como administrador y lanzar los comandos que allí se encuentran, dando como resultado lo siguiente:
![](_attachments/Pasted%20image%2020240423135243.png)
en donde **outcome=success** nos indica el correcto funcionamiento de los comandos.

Cabe añadir que el resto de cambios y particularidades del POM.xml que logré encontrar se encuentran documentadas en el mismo xml.

---

### Ejercicio 3
*"Construya el mismo (provisione y deploye)  e invoque (desde curl) los los endpoint ofrecidos por las dos API que contiene el mismo (MensajesApi y RateLimiterConfigApi)"*

A través del comando:
```Bash
mvn package wildfly:dev
```
se provisioná el servidor para poder ofrecer los servicios de la API.

Se ejecutan allí los 2 endpoints siguientes:
![](_attachments/Pasted%20image%2020240423140615.png)
Se activa el Rate Limiter utilizando las creedenciales del usr4, el cual es revisado validado por el Identity Store que allí se encuentra implementado.


Además del previamente mencionado, se ejecuta el endpoint de *MensajeAPI*:
![](_attachments/Pasted%20image%2020240423140807.png)

---
### Ejercicio 4
*" Controlar en la consola que los servicios estén funcionando correctamente."*

Se comprueba el correcto funcionamiento del Identity Store al probar enviar un mensaje con credenciales equivocadas:
![](_attachments/Pasted%20image%2020240423141326.png)
Como se puede observar, estoy recibiendo tanto el 403 de HTTP como el mensaje de *Access Forbidden*, **pero**, además de esto, en el shell del servidor se puede observar como aunque la operación fue rechazada, aún así se consumió un token del bucket del Rate Limiter.

---
### Ejercicio 5
*"Desde JMeter, cargue el Plan de Pruebas PruebaRateLimiter.jmx"*

Se navega hasta la carpeta del repositorio y se abre el archivo de JMeter *"PruebaRateLimiter.jmx"*
![](_attachments/Pasted%20image%2020240423141804.png)

Aquí se encuentran varias cosas previamente configuradas, tanto pruebas, como endpoints, usuarios, gráficas, etc.

---
### Ejercicio 6
*"1. Ejecute el mismo y verifique que el servidor responda los request."*

Se procede a ejecutar las pruebas con los parametros previamente mostrados, cabe destacar que para estas pruebas, desactivé el Rate Limiter para poder contrastar mejor los resultados más adelante.
![](_attachments/Pasted%20image%2020240423143106.png)
En está gráfica se puede observar las request con código HTTP 200 ~~status OK~~ que vienen llegando al servidor, además, se puede ver que las request **nunca** exceden las 5 responses por segundo, esto es debido a como configuramos previamente el testeo funcional.

Si fuésemos a subirle la cantidad de request por segundo que se pueden servir, la gráfica nos quedaría así:
![](_attachments/Pasted%20image%2020240423143358.png)

---
### Ejercicio 7
*"1. Analice el componente Petición Http encargado de generar los request."*

Revisando la *Petición HTTP* de *JMeter* podemos ver varios parametros de configuración que allí se encuentran:
![](_attachments/Pasted%20image%2020240423144019.png)
Entre ellos están:
- Dirección del servidor
- Endpoint a llamar
- Protocolo HTTP a usar
- Número del puerto del servidor
- Parametros del cuerpo de la petición

---
### Ejercicio 10
*"Abra la jconsole disponible en la carpeta bin del servidor y observe el uso de memoria del servidor."*

Utilizando el siguiente comando que encontré en la documentación online de JConsole [aquí](https://docs.oracle.com/en/java/javase/17/management/using-jconsole.html#GUID-0FAC766D-A99A-4E98-B6BC-1D5711C71CAE)
```Bash
jconsole $PIDdelServidor
```

Abrimos una instancia de JConsole que registre los datos de utilización de recursos del servidor:
![](_attachments/Pasted%20image%2020240423145138.png)
Como muestra la imagen, podemos ver una miriada de datos acerca del uso de recursos del servidor, pero por ahora lo que nos interesa es la memoria, en la imagen previa se visualiza como consume el servidor en standby.

Si fuesemos a hacer el mismo test funcional que utilizamos con JMeter, veremos como cambian los valores de la memoria:
![](_attachments/Pasted%20image%2020240423150012.png)
Podemos ver, aparte del típico comportamiento de la memoria de cualquier servidor que utilize Garbage Collection, que tan pronto empezamos a hacer peticiones al servidor, la memoria de heap utilizada por el servidor se empieza a disparar, junto con el uso de threads de CPU.

En esta imagen se evidencia mejor este comportamiento:
![](_attachments/Pasted%20image%2020240423150320.png)
Notesé como se disparan las gráficas.


Una vez terminadas las peticiones, el uso de memoria vuelve a su estádo de standby:
![](_attachments/Pasted%20image%2020240423150503.png)

---
# Parte C
### Ejercicio 1
*"Utilizando el endpoint de configuración (analice el código para repasar como funciona), active el RateLimiter del lado del servidor."*

Utilizando el endpoint de curl que se encuentra en el archivo: *RateLimiterConfigApi.java*
podemos activar/desactivar el Rate Limiter del servidor, aquí se evidencia su activación:
![](_attachments/Pasted%20image%2020240423173120.png)

---
### Ejercicio 2
*"1. Ejecute nuevamente el test de carga y observe los resultados. Lo mostrado por la gráfica de “Response code per second” debería de tener correlación con la configuración del RateLimiter."*

Volvemos a ejecutar el testeo de JMeter que previamente hemos utilizado, pero está vez haciendo uso del Rate Limiter configurado tal cual viene en el repositorio:
![](_attachments/Pasted%20image%2020240423173423.png)
Esta vez podemos observar 2 funciones distintas, la roja indica la cantidad de respuestas con status HTTP 200 ~~osease estádo OK~~ y la otra indica la cantidad de respuestas con status HTTP 429 ~~osease, demásiadas peticiones~~


Se puede observar que el servidor está solo siendo capaz de servir 2 peticiones por segundo, estos datos se encuentran en oposición a lo redactado en la documentación del archivo *RateLimiter.java*. 
En donde se detalla que la implementación específica del limitador tipo balde, debería ~~teóricamente hablando~~, ser capaz de manejar 1 petición por segundo.
![](_attachments/Pasted%20image%2020240423175407.png)

---

# Parte D
### Ejercicio 1
*" ¿Que tipo de RateLimiter está implementando?"*

Tal y como se discutió previamente en este documento, el limitador de peticiones que se encuentra actualmente implementado es del tipo balde, en resumidas cuentas, el servidor genera X cantidad de tokens cada cierto tiempo, y cada vez que se le hace una petición al servidor, este consume uno de los tokens; este balde tiene un limite de cuantos tokens puede tener almacenado en un momento dado.

La alegoría con un balde viene en parte al mecánismo convencional de un balde lleno de agua, cada vez que se utiliza el agua del balde, este mismo disminuye, pero con el tiempo se va rellenando ~~ya sea artificalmente con una mangera, o naturalmente a través del proceso de que caiga agua del cielo~~

---
### Ejercicio 2
*"Cambie la configuración del mismo para que en lugar de procesar dos tps, procese 4 tps. Además aumente al doble la capacidad inicial de bucket."*

Se cambia la configuración del balde localizada en *RateLimiter.java* para que satisfaga la necesidad de servir 4 transacciones por segundo, además de doblar la capacidad total del balde.
El código resultante es el siguiente:
![](_attachments/Pasted%20image%2020240423181659.png)

---
### Ejercicio 3
*"Ejecute nuevamente el Plan de Pruebas y verifique que el comportamiento del lado del cliente es el esperado."*

Se vuelve a ejecutar el testeo funcional, y ahora se puede observar como las gráficas han cambiado, la función que detalla la cantidad de respuestas status 200 por segundo se encuentra estable en 4.
También es importante apreciar como el cambiar la cantidad máxima del balde nos permite ver como este influye en cuanto tarde el Rate Limiter en hacer efecto, evidenciado por una mayor cantidad de segundos en donde las peticiones de status 200 no fuerón limitadas por el Rate Limiter.

![](_attachments/Pasted%20image%2020240423181938.png)

---
# Parte E
### Ejercicio 1
*"Analice la implementación del Identity Store en memoria."*

Reviso el código del Identity Store en memoria, y me encuentro con la anotación *@Vetoed*
de acuerdo al javadoc proporcionado por Eclipse, la anotación evita que la clase se cargue al contenedor.

Comento el @Vetoed del Identity Store en memoria, y des-comento el que se encuentra en el IdentityStore de hibernate.

Tras esto reviso el código, en resumen, tengo un hashmap<NombreUsuario,ObjetoUsuarioSistema> en donde se guardan los usuarios del Identity Store, en este caso hay 4 usuarios, con diferentes roles cada uno.
![](_attachments/Pasted%20image%2020240423190612.png)


Tras esto está la implementación del sistema de validación, en donde conseguimos el nombre y contraseña **en texto plano** de el usuario a ser validado, inmediatamente, nos fijamos si existe ese usuario a validar dentro de nuestro hashmap de usuarios del Identity Store, y checkeamos que la contraseña sea la misma, si es así, devolvemos un *CredentialValidationResult* con el nombre del usuario + el ArrayList de los roles a los que pertenece.
![](_attachments/Pasted%20image%2020240423191104.png)

---
### Ejercicio 2
*"¿Qué cambios habría que hacer para que el usr1 pueda también invocar el endpoint de configuración del RateLimiter?"*

En lo personal, yo hipotetizo 2 posibles caminos, el primero y más seguro al mi parecer, es cambiar el rol de usr1 a "admin" dentro del hashmap del Identity Store en memoria, la otra manera sería cambiar el rol necesario para poder configurar el Rate Limiter.

Para este ejemplo prové la primera hipótesis:
![](_attachments/Pasted%20image%2020240423192426.png)

Y aquí el resultado confirmando mi hipótesis:
![](_attachments/Pasted%20image%2020240423191906.png)

---
### Ejercicio 5
"1 *Realice los cambios necesarios para permitirle a un usuario usr5, poder acceder a la api de configuración.(¿son cambios en el código o en algún otro lugar?)*"

Para poder añadir un usr5 al validador con DB, primero hay que dejarlo configurado como estába previamente, con *@Vetoed* para el validador en memoria, y des-comentando esa anotación para el validador de DB.

Tras esto, me dirijo al *persistance.xml* para averiguar donde se están guardando las entradas iniciales de las DB, dentro del mismo hallo donde se guardan los usuarios y contraseñas de los mismos, dentro de *initial_data.sql*

Añado las entradas correspondientes para que usr5 exista en la DB, ayudandome con el hash generado por la clase: *HashFunctionUtil.java*
![](_attachments/Pasted%20image%2020240423200432.png)

Aquí el código que utilicé para hashear la contraseña de usr5:
![](_attachments/Pasted%20image%2020240423195911.png)


Y finalmente, pruebo la veracidad del funcionamiento del código:
![](_attachments/Pasted%20image%2020240423201025.png)

Como se puede observar, la implementación con DB me levantó el usr5 como admin, dejandome así poder activar el Rate Limiter
# Referencias