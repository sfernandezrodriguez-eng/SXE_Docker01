# Práctica Docker: Gestión de Contenedores e Imágenes Alpine

En este repositorio se documentan los pasos realizados para la práctica con Docker, utilizando la imagen liviana de Alpine Linux.


---

## 1. Descarga de la imagen Alpine (versión fijada)

Para evitar errores en el entorno, se fijó una versión explícita en lugar de utilizar la etiqueta latest, en este caso la 3.22.

### Comandos ejecutados:

docker pull alpine:3.22
docker images

### Resultado y observación:
La imagen se descargó correctamente desde el repositorio oficial de Docker Hub.

Al ejecutar docker images, comprobamos que la imagen alpine:3.22 con ID c83674e19990 ocupa únicamente 8.3 MB en disco.

---

## 2. Creación de contenedor sin nombre y sin arrancar

Se creó un contenedor a partir de la imagen descargada sin asignarle un nombre personalizado y sin iniciarlo.

### Comandos ejecutados:

docker create alpine:3.22
docker ps -a

### Respuestas a las preguntas:
**¿En qué estado queda?** El contenedor queda en estado Created.

**¿Qué nombre le ha puesto Docker?** Docker asignó automáticamente un nombre aleatorio compuesto por un adjetivo y el apellido de una figura científica destacada. En este caso, le asignó el nombre nice_roentgen.

---

## 3. Creación y arranque del contenedor dam_alp1

Se creó y arrancó un nuevo contenedor denominado dam_alp1 ejecutando una shell interactiva. (Se cometió un error a la hora de crearlo y se llamó dam_al1, posteriormente se renombró mediante docker rename dam_al1 dam_alp1, pero si en alguna captura o registro sale así el nombre es por eso).

### Comando ejecutado:

docker run -it --name dam_alp1 alpine:3.22 /bin/sh

### ¿Qué opciones necesitas para poder escribir dentro?
**-i --interactive**: Mantiene abierto el canal de entrada estándar (STDIN), lo que permite enviar comandos al contenedor.

**-t --tty**: Asigna una pseudo-terminal (TTY) que emula la interfaz de consola interactiva.

La combinación **-it** permite interactuar directamente con la shell /bin/sh.

---

## 4. Inspección de red e IP dentro del contenedor

Desde el interior de la terminal de dam_alp1, se comprobó la dirección IP asignada y la conectividad a Internet.

### Comandos ejecutados (dentro del contenedor):

ip a
ping -c 4 google.com

### Resultados:
**Dirección IP:** La interfaz eth0 recibió la IP 172.17.0.2/16.

**Ping a google.com:** Se enviaron y recibieron 4 paquetes con un 0% de pérdida de paquetes, confirmando acceso a la red externa.

---

## 5. Comunicación entre contenedores (dam_alp1 y dam_alp2)

Se dejó dam_alp1 en marcha y se creó dam_alp2 en segundo plano. Luego se realizaron pruebas de ping de uno a otro.

### Comandos ejecutados:

docker run -d -it --name dam_alp2 alpine:3.22 /bin/sh
docker exec -it dam_alp1 ping -c 3 172.17.0.3
docker exec -it dam_alp1 ping -c 3 dam_alp2

### Explicación de los resultados:
**Ping por IP (172.17.0.3):** Funciona. Ambos contenedores están en la misma red bridge por defecto y pueden comunicarse directamente a nivel IP.

**Ping por nombre (dam_alp2):** Falla (ping: bad address 'dam_alp2'). La red bridge por defecto de Docker no incluye resolución DNS automática por nombre de contenedor.

---

## 6. Monitoreo del consumo de memoria

Se comprobó cuánta memoria consumen ambos contenedores con los dos en marcha.

### Comandos y respuestas:
**¿Hay un comando de Docker para eso?** Sí, el comando es docker stats.

docker stats --no-stream

**Resultado:** Tanto dam_alp1 como dam_alp2 consumen aproximadamente 512 KiB de memoria RAM cada uno (un 0.00% del límite del sistema de 15.39 GiB).

---

## 7. Parada de contenedores y estado posterior

Al salir con exit o detener los contenedores mediante comando, se analizó el cambio de estado.

### Comandos ejecutados:

docker stop dam_alp1 dam_alp2
docker ps -a
docker stats --no-stream

### Respuestas a las preguntas:
**¿Qué les ha pasado?** Los contenedores pasan al estado Exited.

**¿Qué ves ahora y por qué?**

En docker ps -a aparecen listados con estado Exited (0) o Exited (137).

Al repetir docker stats, ya no se muestran métricas de uso porque los contenedores están detenidos.

**Explicación:** Un contenedor se mantiene activo únicamente mientras su proceso principal (/bin/sh) esté en ejecución. Al salir de la shell o detenerlo, el proceso finaliza y el contenedor se apaga.

---

## 8. Análisis del uso de disco (Imágenes vs Contenedores)

Se analizó el espacio ocupado en disco diferenciando entre imágenes y contenedores.

### Comando ejecutado:

docker system df -v

### Distinción entre imágenes y contenedores:
**Imágenes (Images space usage):** Contienen el sistema de archivos base inmutable. La imagen alpine:3.22 ocupa 8.3 MB en disco.

**Contenedores (Containers space usage):** Solo guardan los cambios realizados en su capa de lectura/escritura. dam_alp1 ocupa 167 B, dam_alp2 ocupa 5 B y nice_roentgen ocupa 0 B.