"# mensajeria" 
# Evaluación Técnica - Microservicios de Geolocalización

## Objetivo

Construir una solución para geocodificar direcciones, implementando una arquitectura basada en microservicios. La solución debe ejecutarse completamente en contenedores Docker.

## Descripción

La solución debe constar de dos servicios que se comunican utilizando un sistema de mensajería.

- **API GEO**: Exponer un endpoint que reciba una dirección y almacene la solicitud en una base de datos. Debe enviar la información a otro microservicio para la geocodificación.
- **Geocodificador**: Recibir la solicitud y utilizar la API de OpenStreetMap para obtener coordenadas geográficas. Enviar el resultado de vuelta a la API GEO a través del sistema de mensajería.

## Operatoria

1. **API GEO**
   - Exponer un endpoint:
     ```http
     POST /geolocalizar
     ```
     **Request Body:**
     ```json
     {
       "calle": "",
       "numero": "",
       "ciudad": "",
       "codigo_postal": "",
       "provincia": "",
       "pais": ""
     }
     ```
     **Response:**
     ```json
     {
       "id": "<UUID>"
     }
     ```
     - Guardar la petición en la base de datos con estado **PROCESANDO**.
     - Enviar la información a la cola de mensajería (RabbitMQ, Kafka, etc.).

2. **Geocodificador**
   - Leer el mensaje de la cola de mensajería.
   - Utilizar la API de OpenStreetMap para obtener coordenadas.
   - Enviar el resultado de vuelta a la API GEO.

3. **API GEO** (Actualización de estado)
   - Recibir la respuesta del Geocodificador.
   - Actualizar el estado en la base de datos a **TERMINADO** con latitud y longitud.
   - Exponer un segundo endpoint para consultar el estado:
     ```http
     GET /geocodificar?id=<UUID>
     ```
     **Response:**
     ```json
     {
       "id": "<UUID>",
       "latitud": xxxx,
       "longitud": xxxx,
       "estado": "PROCESANDO | TERMINADO"
     }
     ```

## Requisitos Tecnológicos

- **Framework**: .NET Core 3.1 o superior (recomendado .NET 8).
- **Mensajería**: RabbitMQ, Apache Kafka u otra compatible con Docker.
- **Base de datos**: SQL Server, MySQL o MongoDB.
- **Contenedores**: Docker con un archivo `docker-compose.yml` para orquestar los servicios.
- **Repositorio**: GitHub con documentación clara y estructurada.

## Forma de Trabajo

1. **Estructurar el proyecto** en un repositorio GitHub con una arquitectura modular.
2. **Configurar Docker Compose** para levantar los servicios con una sola instrucción.
3. **Desplegar la API GEO y el Geocodificador** asegurando la comunicación a través del sistema de mensajería.
4. **Probar la solución** utilizando herramientas como Postman o cURL.
5. **Escribir documentación** para el uso del sistema y los comandos necesarios para su despliegue.

---

## Ejemplo de `docker-compose.yml`
```yaml
version: '3.9'

services:
  api-geo:
    build: ./ApiGeo
    ports:
      - "8080:80"
    depends_on:
      - db
      - rabbitmq

  geocodificador:
    build: ./Geocodificador
    depends_on:
      - rabbitmq

  db:
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=geo
    ports:
      - "3306:3306"

  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
```

## Comandos de Ejecución

1. **Clonar el repositorio:**
   ```sh
   git clone https://github.com/tu-usuario/tu-repo.git
   cd tu-repo
   ```

2. **Construir y ejecutar los contenedores:**
   ```sh
   docker-compose up --build
   ```

3. **Probar la API:**
   ```sh
   curl -X POST http://localhost:8080/geolocalizar -H "Content-Type: application/json" -d '{
       "calle": "Av. Las Heras",
       "numero": "1234",
       "ciudad": "Buenos Aires",
       "codigo_postal": "C1018",
       "provincia": "Buenos Aires",
       "pais": "Argentina"
   }'
   ```

4. **Consultar el estado:**
   ```sh
   curl "http://localhost:8080/geocodificar?id=<UUID>"
   ```

---