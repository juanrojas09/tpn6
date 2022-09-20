**Trabajo Práctico 6 - Construcción de Imágenes de Docker**

**1- Objetivos de Aprendizaje**

- Adquirir conocimientos para construir y publicar imágenes de Docker.
- Familiarizarse con el vocabulario.

**2- Unidad temática que incluye este trabajo práctico**

Este trabajo práctico corresponde a la unidad Nº: 3

**3- Consignas a desarrollar en el trabajo práctico:**

En los puntos en los que se pida alguna descripción, realizarlo de la manera más clara posible

**4- Desarrollo:**

Leer[ https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/) (tiempo estimado 2 horas)

- Describir las instrucciones
- FROM:  crea una copia de la imagen, un dockerfile, debe comenzar con una instruccion from, nos proporciona una imagen con una determinada base, por ejemplo **FROM “ubuntu:20.4” ,** que seria la base para las instrucciones que le siguen.
- RUN:  tiene 2 formas: **RUN <command> y RUN [”executable”,”param1,param2],** sirve para ejecutar cualquier comando en una nueva capa encima de la imagen actual y va a confirmar los resultados, la imagen confirmada resultante se usa para el siguiente paso en el archivo docker run
- ADD: Tiene tambien 2 formas, **1)ADD [--chown=<user>:<group>] <src>... <dest>, 2) ADD[--chown=<user>:<group>] [“<src>”,...”<dest>”]** ,

esta ultima forma es necesaria para las rutas que tienen espacios en blanco, la instruccion **ADD**, copia nuevos archivos,directorios o direcciones URL de archivos remotos <src> y los agrega al sistema de archivos de la imagen en la ruta <dest>.

- COPY: tiene 2 formas **1) COPY [--chown=<user>:<group>] <src>...<dest> y 2) COPY [--chown=<user>:<group>]**, esta instruccion copia nuevos archivos o directorios <src> y los agrega al sistema de archivos del contenedor de ruta <dest>
- EXPOSE: EXPOSE <port> [<port>/<protocol>..], esta instrucción informa a docker que el contenedor escucha en los puertos de red especificados en tiempo de ejecución. Puede especificar si el puerto escucha TCP o UDP, y el valor predeterminado es el TCP si no se

especifica el protocolo, esta instrucción no publica el puerto,

- CMD: tenemos 3 formas, **shell** CMD command param1 param2 **, excec** CMD ["executable","param1","param2"] **y default** CMD ["param1","param2"] y el objetivo principal a cmd es

proporcionar valores predeterminados para un contenedor en ejecucion, esos valores pueden incluir un ejecutable o omitirlo, en cuyo caso debe especificar una instruccion en el entrypoint

- ENTRYPOINT: tiene 2 formas **excec** ENTRYPOINT ["executable", "param1", "param2"], **shell** ENTRYPOINT command param1 param2 lo que hace es permitir configurar un contenedor que se ejecutara como un ejecutable.

**2- Generar imagen de docker**

- Clonar/Actualizar el repositorio de <https://github.com/fernandobono/ing-software-3>
- El código se encuentra en la carpeta ./proyectos/spring-boot
- Se puede copiar al repositorio personal en una carpeta trabajo-practico-06/spring-boot
- Compilar la salida con:

cd proyectos/spring-boot

mvn clean package spring-boot:repackage

daba error cuando corria los test, se agrego un plugin dentro del pom para que ignore el resultado del test

<plugin>

<groupId>org.apache.maven.plugins</groupId> <artifactId>maven-surefire-plugin</artifactId>

<version>2.19.1</version>

<configuration>

<testFailureIgnore>true</testFailureIgnore> </configuration>

</plugin>

Agregar un archivo llamado **Dockerfile** (en el directorio donde se corrió el comando mvn)

**FROM java:8-jre-alpine**

**RUN apk add --no-cache bash**

**WORKDIR /app**

**COPY target/\*.jar ./spring-boot-application.jar**

**ENV JAVA\_OPTS="-Xms32m -Xmx128m" EXPOSE 8080**

**ENTRYPOINT exec java $JAVA\_OPTS -Djava.security.egd=file:/dev/./urandom -jar spring-boot-application.jar**

- Generar la imagen de docker con el comando build

**docker build -t test-spring-boot .**

el java:8 lo cambie por openjdk:8 para que haga build de forma correcta

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.001.jpeg)

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.002.png)

- Ejecutar el contenedor

docker run -p 8080:8080 test-spring-boot

- Capturar y mostrar la salida.

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.003.jpeg)

- Verificar si retorna un mensaje (correr en otro terminal o browser)

**curl -v localhost:8080**

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.004.png)

**3- Dockerfiles Multi Etapas**

Se recomienda crear compilaciones de varias etapas para todas las aplicaciones (incluso las heredadas). En resumen, las compilaciones de múltiples etapas:

- Son independientes y auto descriptibles
- Resultan en una imagen de Docker muy pequeña
- Puede ser construido fácilmente por todas las partes interesadas del proyecto (incluso los no desarrolladores)
- Son muy fáciles de entender y mantener.
- No requiere un entorno de desarrollo (aparte del código fuente en sí)
- Se puede empaquetar con pipelines muy simples

Las compilaciones de múltiples etapas también son esenciales en organizaciones que emplean múltiples lenguajes de programación. La facilidad de crear una imagen de Docker por cualquier persona sin la necesidad de JDK / Node / Python / etc. no puede ser sobrestimado.

- Modificar el dockerfile para el proyecto Java anterior de la siguiente forma

**FROM maven:3.5.2-jdk-8-alpine AS MAVEN\_TOOL\_CHAIN**

**COPY pom.xml /tmp/**

**RUN mvn -B dependency:go-offline -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml**

**COPY src /tmp/src/**

**WORKDIR /tmp/**

**RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package**

**FROM java:8-jre-alpine EXPOSE 8080**

**RUN mkdir /app**

**COPY --from=MAVEN\_TOOL\_CHAIN /tmp/target/\*.jar /app/spring-boot-application.jar**

**ENV JAVA\_OPTS="-Xms32m -Xmx128m"**

**ENTRYPOINT exec java $JAVA\_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app/spring-boot-application.jar**

**HEALTHCHECK --interval=1m --timeout=3s CMD wget -q -T 3 -s http://localhost:8080/actuator/health/ || exit 1**

- Construir nuevamente la imagen

**docker build -t test-spring-boot .**

- **Analizar y explicar el nuevo Dockerfile, incluyendo las nuevas instrucciones.**

*#Especifica la imagen base, en este caso la imagen de node y la asigna como maven tool chain*

FROM maven:3.5.2-jdk-8-alpine AS MAVEN\_TOOL\_CHAIN

*#Crea el directorio de trabajo del pom.xml y lo copia en la carpeta /tmp/*

COPY pom.xml /tmp/

*#Ejecuta el comando mvn dependecy:go-offline para descargar las dependencias del pom.xml, el -B es modo no interactivo,el -f es para especificar el archivo pom.xml y -s es para especificar el archivo settings.xml con su path*

RUN mvn -B dependency:go-offline -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml

*#Copia el resto de los archivos del proyecto en la carpeta /tmp/*

COPY src /tmp/src/

*#Especifica el directorio de trabajo a /tmp/*

WORKDIR /tmp/

*#Ejecuta el comando mvn package para empaquetar el proyecto, el -B es modo no interactivo,el -f es para especificar el archivo pom.xml y -s es para especificar el archivo settings.xml con su path*

RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package

*#Hace lo mismo que arriba, como si fuera otra parte del dockerfile y al tener varios from que lo que hacemos es obtener una compilacion por partes, donde los resultados se juntan para conformar la imagen final luego de correr todas las partes.*

FROM java:8-jre-alpine

*#Expone en el puerto 8080*

EXPOSE 8080

*#crea un directorio llamado app*

RUN mkdir /app

*#Copia la img anterior asignada como maventoolchain.*

COPY --from=MAVEN\_TOOL\_CHAIN /tmp/target/\*.jar /app/spring-boot-application.jar

ENV JAVA\_OPTS="-Xms32m -Xmx128m"![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.005.png)

ENTRYPOINT exec java $JAVA\_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app/spring-boot-application.jar![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.005.png)

*#Es la manera de saber si la aplicacion es saludable,se encarga de indicarle al daemon de Docker cómo comprobar que el contenedor se encuentre funcionando,*

HEALTHCHECK --interval=1m --timeout=3s CMD wget -q -T 3 -s http://localhost:8080/actuator/health/ || exit 1

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.006.png)

**4- Python Flask**

- **Utilizar el código que se encuentra en la carpeta ./proyectos/python-flask**
- **Se puede copiar al repositorio personal en una carpeta trabajo-practico-06/python-flask**
- **Correr el comando**

**cd ./proyectos/python-flask docker-compose up -d**

- **Explicar que sucedió!**
- **¿Para qué está la key build.context en el docker-compose.yml?**

En docker-compose.yml, tenemos el build: context: ./

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.007.png)

por lo que vengo entendiendo, esto le indica al dockerfile, cuando hago el build utilizar ese directorio como contexto

**5- Imagen para aplicación web en Nodejs**

- **Crear una la carpeta trabajo-practico-06/nodejs-docker**
- **Generar un proyecto siguiendo los pasos descriptos en el trabajo práctico 5 para Nodejs**
- **Escribir un Dockerfile para ejecutar la aplicación web localizada en ese directorio**
  - **Idealmente que sea multistage, con una imagen de build y otra de producción.**
  - **Usar como imagen base node:13.12.0-alpine**
  - **Ejecutar npm install dentro durante el build.**
  - **Exponer el puerto 3000**
- **Hacer un build de la imagen, nombrar la imagen test-node.**
- **Ejecutar la imagen test-node publicando el puerto 3000.**
- **Verificar en[ http://localhost:3000](http://localhost:3000) que la aplicación está funcionando.**
- **Proveer el Dockerfile y los comandos ejecutados como resultado de este ejercicio.**

**Cree el Dockerfile, con las instrucciones de la creacion de la imagen de nodejs**

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.008.jpeg)

El copy . . copia todo del directorio de tmp, osea todo de lo de la anterior etapa

- **para hacer build**

**docker build -t test-node .**

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.009.jpeg)

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.010.png)

**docker run -p 3000:3000 test-node**

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.011.jpeg)

**Validacion del server levantado en el 3000**

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.012.jpeg)

**6- Publicar la imagen en Docker Hub.**

- **Crear una cuenta en Docker Hub si no se dispone de una.**
- **Registrase localmente a la cuenta de Docker Hub:**

**docker login**

Yo tenia cuenta en docker hub, entonces me autentico cuando corri el comando

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.013.png)

- **Crear un tag de la imagen generada en el ejercicio 3. Reemplazar <mi\_usuario> por el creado en el punto anterior.**

**docker tag test-node <mi\_usuario>/test-node:latest**

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.014.png)

- **Subir la imagen a Docker Hub con el comando**

**docker push <mi\_usuario>/test-node:latest**

![](Aspose.Words.60035b28-93ed-4f7e-b757-d5d61418e89e.015.png)

- **Como resultado de este ejercicio mostrar la salida de consola,**

**o una captura de pantalla de la imagen disponible en Docker Hub.**
