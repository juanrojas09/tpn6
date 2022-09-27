#Especifica la imagen base, en este caso la imagen de node y la asigna como maven tool chain
FROM maven:3.5.2-jdk-8-alpine AS MAVEN_TOOL_CHAIN
#Crea el directorio de trabajo del pom.xml y lo copia en la carpeta /tmp/
COPY pom.xml /tmp/
#Ejecuta el comando mvn dependecy:go-offline para descargar las dependencias del pom.xml, el -B es modo no interactivo,el -f es para especificar el archivo pom.xml y -s es para especificar el archivo settings.xml con su path
RUN mvn -B dependency:go-offline -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml
#Copia el resto de los archivos del proyecto en la carpeta /tmp/
COPY src /tmp/src/
#Especifica el directorio de trabajo a /tmp/
WORKDIR /tmp/
#Ejecuta el comando mvn package para empaquetar el proyecto, el -B es modo no interactivo,el -f es para especificar el archivo pom.xml y -s es para especificar el archivo settings.xml con su path
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package
#Hace lo mismo que arriba, como si fuera otra parte del dockerfile y al tener varios from que lo que hacemos es obtener una compilacion por partes, donde los resultados se juntan para conformar la imagen final luego de correr todas las partes.
FROM openjdk:8-jre-alpine
#Expone en el puerto 8080
EXPOSE 8080
#crea un directorio llamado app
RUN mkdir /app
#Copia la img anterior asignada como maventoolchain.
COPY --from=MAVEN_TOOL_CHAIN /tmp/target/*.jar /app/spring-boot-application.jar

ENV JAVA_OPTS="-Xms32m -Xmx128m"

ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app/spring-boot-application.jar
#Es la manera de saber si la aplicacion es saludable,se encarga de indicarle al daemon de Docker cómo comprobar que el contenedor se encuentre funcionando, 
HEALTHCHECK --interval=1m --timeout=3s CMD wget -q -T 3 -s http://localhost:8080/actuator/health/ || exit 1
