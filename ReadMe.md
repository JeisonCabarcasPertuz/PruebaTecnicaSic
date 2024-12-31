# Proyecto: App y Sicapp con Docker

Este proyecto contiene dos aplicaciones:
- **Backend (app)**: Hecho con Spring Boot 3.3.7 y Java 21.
- **Frontend (sicapp)**: Hecho con React, TypeScript y Vite.

Ambas aplicaciones se configuran y ejecutan juntas utilizando Docker.

## Requisitos previos

Antes de comenzar, asegúrate de tener instalados:

- [Docker](https://www.docker.com/) (versión 20.10 o superior).
- [Docker Compose](https://docs.docker.com/compose/) (versión 1.29 o superior).

## Estructura del proyecto

La estructura del proyecto es la siguiente:

```
project/
├── app/
│   ├── Dockerfile
│   └── target/
│       └── app-1.0.0.jar
├── sicapp/
│   ├── Dockerfile
│   ├── .env
│   └── src/
├── docker-compose.yml
```

## Configuración del backend (app)

El backend está configurado para ejecutarse en el puerto `8080` y conectarse a una base de datos PostgreSQL.

### Archivo `app/Dockerfile`
```dockerfile
# Usar la imagen oficial de OpenJDK para Java 21
FROM openjdk:21-jdk-slim

# Crear directorio de trabajo
WORKDIR /app

# Copiar el archivo JAR generado
COPY target/app-1.0.0.jar app.jar

# Exponer el puerto en el que corre la aplicación
EXPOSE 8080

# Comando para ejecutar la aplicación
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Configuración del frontend (sicapp)

El frontend está configurado para ejecutarse en el puerto `5173`.

### Archivo `sicapp/Dockerfile`
```dockerfile
# Usar la imagen oficial de Node.js
FROM node:18

# Crear directorio de trabajo
WORKDIR /app

# Copiar archivos del proyecto
COPY . .

# Instalar dependencias
RUN npm install

# Construir la aplicación
RUN npm run build

# Usar una imagen de servidor web para servir el contenido estático
FROM nginx:alpine
COPY --from=0 /app/dist /usr/share/nginx/html

# Exponer el puerto en el que corre el servidor
EXPOSE 80

# Comando para iniciar NGINX
CMD ["nginx", "-g", "daemon off;"]
```

### Archivo `sicapp/.env`

Configura las variables de entorno necesarias para el frontend:
```env
VITE_BACK_BASE_URL=http://backend:8080
VITE_API_USERNAME=tu_usuario
VITE_API_PASSWORD=tu_contraseña
```

## Configuración de Docker Compose

El archivo `docker-compose.yml` orquesta los contenedores para el backend, frontend y la base de datos PostgreSQL.

### Archivo `docker-compose.yml`
```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./app
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/sicregistros
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: Ogata@1995
    depends_on:
      - db

  frontend:
    build:
      context: ./sicapp
      dockerfile: Dockerfile
    ports:
      - "5173:80"
    environment:
      VITE_BACK_BASE_URL: http://backend:8080

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: sicregistros
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: Ogata@1995
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

## Pasos para ejecutar el proyecto

1. Asegúrate de que el archivo JAR del backend esté en la ruta `app/target/app-1.0.0.jar`.

2. Configura las variables necesarias en `sicapp/.env`.

3. Desde la raíz del proyecto, ejecuta los siguientes comandos:

   ```bash
   # Construir y levantar los contenedores
   docker-compose up --build
   ```

4. Verifica que los servicios estén corriendo:
   - Backend: [http://localhost:8080](http://localhost:8080)
   - Frontend: [http://localhost:5173](http://localhost:5173)

## Notas adicionales

- Si necesitas detener los contenedores, ejecuta:
  ```bash
  docker-compose down
  ```

- Los datos de la base de datos se guardan en el volumen `db_data`, que persiste incluso si los contenedores se detienen.

- Si realizas cambios en el código fuente, vuelve a construir los contenedores:
  ```bash
  docker-compose up --build
  ```

---

¡Listo! Ahora tienes un entorno completamente configurado para ejecutar tu proyecto con Docker.

