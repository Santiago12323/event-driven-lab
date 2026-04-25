# Event-Driven Microservices Lab

## Spring Boot + RabbitMQ + Docker Compose

---

## 1. Descripción General

Este laboratorio tiene como objetivo implementar una arquitectura basada en eventos utilizando microservicios desarrollados con Spring Boot. La comunicación entre servicios se realiza de forma asíncrona mediante RabbitMQ como sistema de mensajería.

El sistema está completamente dockerizado y orquestado mediante Docker Compose, permitiendo su despliegue en entornos locales o plataformas online como Codespaces o Killercoda.

---

## 2. Objetivos

* Implementar comunicación asíncrona entre microservicios
* Utilizar RabbitMQ como broker de mensajes
* Contenerizar servicios con Docker
* Orquestar múltiples servicios con Docker Compose
* Ejecutar el sistema en un entorno cloud gratuito

---

## 3. Arquitectura del Sistema

El sistema sigue un modelo **Event-Driven Architecture (EDA)**:

```
Cliente → Producer → Exchange → Queue → Consumer
```

###  Flujo detallado

1. El cliente envía una petición HTTP al Producer
2. El Producer publica un mensaje en un Exchange de RabbitMQ
3. El Exchange enruta el mensaje hacia una Queue
4. El Consumer escucha la Queue
5. El Consumer procesa el mensaje

---

##  4. Estructura inicail del Proyecto

```
event-driven-lab/
├── producer-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
├── consumer-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
├── .devcontainer/
│   └── devcontainer.json
├── docker-compose.yml
└── README.md
```

---

##  5. Tecnologías Utilizadas

* Java 17
* Spring Boot
* Spring AMQP (RabbitMQ)
* Maven
* Docker
* Docker Compose
* GitHub Codespaces

---

## 6. Configuración del Entorno (Codespaces)

Se utiliza un contenedor de desarrollo definido en `.devcontainer/devcontainer.json`.

### Características:

* Java 17 preinstalado
* Maven configurado
* Docker-in-Docker habilitado
* Puertos expuestos: 8080, 8081, 15672, 5672

Esto permite desarrollar y ejecutar todo el laboratorio directamente en la nube.

---

##  7. Servicio Productor

###  Función

Encargado de enviar mensajes a RabbitMQ mediante una API REST.

---

###  Configuración

Archivo: `application.properties`

```
server.port=8080

spring.rabbitmq.host=rabbitmq
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

app.rabbitmq.exchange=messages.exchange
app.rabbitmq.queue=messages.queue
app.rabbitmq.routingkey=messages.routingkey
```

---

###  Componentes

#### 1. RabbitMQConfig

Define:

* Queue (cola)
* Exchange (tipo direct)
* Binding (relación entre ambos)

---

#### 2. MessageController

Endpoint:

```
POST /api/messages/send?message=texto
```

Envía mensajes al broker usando RabbitTemplate.

---

##  8. Servicio Consumidor

###  Función

Escuchar mensajes desde RabbitMQ y procesarlos.

---

###  Configuración

```
spring.rabbitmq.host=rabbitmq
spring.rabbitmq.port=5672

app.rabbitmq.queue=messages.queue
```

---

###  Componentes

#### 1. RabbitMQConfig

Define la cola para asegurar su existencia.

---

#### 2. MessageListener

```
@RabbitListener(queues = "messages.queue")
```

Procesa automáticamente los mensajes recibidos.

---

##  9. Dockerización

###  Producer Dockerfile

```
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

---

###  Consumer Dockerfile

```
FROM eclipse-temurin:17-jre-jammy
WORKDIR /app
COPY target/*.jar app.jar
ENTRYPOINT ["java","-jar","app.jar"]
```

---

## 10. Docker Compose

Archivo principal que orquesta los servicios:

```
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:management
    hostname: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"

  producer:
    image: TU_USUARIO/producer-service
    ports:
      - "8080:8080"
    depends_on:
      - rabbitmq

  consumer:
    image: TU_USUARIO/consumer-service
    depends_on:
      - rabbitmq
```

---

##  11. Ejecución del Proyecto

### 1. Clonar repositorio

```
git clone https://github.com/tu-usuario/event-driven-lab.git
cd event-driven-lab
```

---

### 2. Construir servicios

```
mvn clean package
```

---

### 3. Construir imágenes

```
docker build -t producer-service ./producer-service
docker build -t consumer-service ./consumer-service
```

---

### 4. Subir a Docker Hub

```
docker tag producer-service usuario/producer-service
docker push usuario/producer-service

docker tag consumer-service usuario/consumer-service
docker push usuario/consumer-service
```

---

### 5. Levantar servicios

```
docker-compose up -d
```

---

### 6. Verificar estado

```
docker-compose ps
```

---

##  12. Pruebas del Sistema

### Enviar mensaje

```
curl -X POST "http://localhost:8080/api/messages/send?message=HolaMundo"
```

---

### Ver logs del consumidor

```
docker-compose logs consumer
```

Salida esperada:

```
Mensaje recibido: HolaMundo
>>> Mensaje Procesado: HolaMundo
```

---

##  13. Interfaz de RabbitMQ

Acceso:

```
http://localhost:15672
```

Credenciales:

* Usuario: guest
* Contraseña: guest

En la sección **Queues** se puede visualizar el flujo de mensajes.

---


##  14. Estado del Proyecto

✔ Producer funcionando
✔ Consumer funcionando
✔ RabbitMQ operativo
✔ Comunicación asíncrona exitosa
✔ Contenerización completa
✔ Orquestación con Docker Compose

---

Desarrollado por: Santiago Coronado Pinzon
