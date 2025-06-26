# Proyecto Full-Stack con Spring Boot y Angular

Este proyecto es una aplicación web full-stack que utiliza Spring Boot para el backend y Angular para el frontend. La aplicación permite gestionar personas y sus mascotas.

## Tecnologías Utilizadas

- **Backend:**
  - Java 17
  - Spring Boot 3.4.5
  - Maven
  - PostgreSQL
  - Spring Data JPA

- **Frontend:**
  - Angular 19
  - TypeScript
  - HTML
  - SCSS
  - Font Awesome

- **Contenerización y Orquestación:**
  - Docker
  - Docker Compose
  - Kubernetes

## Arquitectura de la Aplicación

La aplicación consta de los siguientes componentes:

- **Frontend:** Una aplicación de cliente desarrollada en Angular.
- **Backend:** Una API RESTful construida con Spring Boot.
- **Base de Datos:** Una instancia de PostgreSQL para la persistencia de datos.

## Despliegue con Kubernetes

Esta sección describe el proceso para desplegar la aplicación web de 3 capas (frontend, backend y base de datos) utilizando Kubernetes.

### Prerrequisitos

Antes de comenzar, se de tener lo siguiente:

- Un clúster de Kubernetes en funcionamiento.
- `kubectl` instalado y configurado para interactuar con tu clúster.
- Las imágenes de Docker para el frontend y el backend (`felipe2p05/angular-client:latest` y `felipe2p05/springboot-app:latest`) disponibles en un registro de contenedores accesible para tu clúster.

### Estructura de los Manifiestos de Kubernetes

El directorio `k8s-manifests/` contiene los siguientes archivos de manifiesto:

- `frontend-deployment.yaml`: Define el despliegue para la aplicación de frontend.
- `frontend-service.yaml`: Expone el frontend a través de un servicio de tipo `NodePort`.
- `backend-deployment.yaml`: Define el despliegue para la aplicación de backend.
- `backend-service.yaml`: Expone el backend a través de un servicio de tipo `ClusterIP`.
- `db-deployment.yaml`: Define el despliegue para la base de datos PostgreSQL.
- `db-service.yaml`: Expone la base de datos a través de un servicio de tipo `ClusterIP`.
- `pgdata-persistentvolumeclaim.yaml`: Define un `PersistentVolumeClaim` para el almacenamiento persistente de la base de datos.

### Despliegue de la Aplicación

1. **Aplicar los manifiestos:**
   Aplica todos los manifiestos de Kubernetes que se encuentran en el directorio `k8s-manifests/` utilizando el siguiente comando:
   ```bash
   kubectl apply -f k8s-manifests/
   ```

2. **Verificar el despliegue:**
   Para asegurar que todos los pods se estén ejecutando correctamente ejecutar:
   ```bash
   kubectl get pods -l 'io.kompose.service in (frontend, backend, db)'
   ```
   También verificar el estado de los servicios y el `PersistentVolumeClaim`:
   ```bash
   kubectl get services -l 'io.kompose.service in (frontend, backend, db)'
   kubectl get pvc -l 'io.kompose.service=pgdata'
   ```

### Acceso a la Aplicación

- **Frontend:**
  El frontend se expone a través de un servicio de tipo `NodePort`. Para acceder a él, se necesita la dirección IP de uno de los nodos del clúster y el puerto asignado por el servicio.
  Obtener la direccion IP y el puerto del nodo con el siguiente comando:
  ```bash
  minikube service frontend --url
  ```
  Luego, accede a la aplicación en tu navegador a través de la dirección de salida `http://<node-ip>:<node-port>`, por ejemplo, `http://192.168.49.2:30248`.

- **Backend:**
  El backend es accesible dentro del clúster a través del servicio `backend` en el puerto `8080`.

- **Base de Datos:**
  La base de datos es accesible dentro del clúster a través del servicio `db` en el puerto `5432`.

### Almacenamiento Persistente

La base de datos PostgreSQL utiliza un `PersistentVolumeClaim` llamado `pgdata` para garantizar que los datos se conserven incluso si el pod de la base de datos se reinicia.

### Limpieza

Para eliminar todos los recursos de la aplicación de tu clúster, ejecuta el siguiente comando:

```bash
kubectl delete -f k8s-manifests/
```

## Despliegue local con Docker Compose

Para levantar la aplicación en un entorno local, es necesario tener instalado Docker y Docker Compose.

1. Clonar el repositorio:

   ```bash
   git clone https://github.com/FepDev25/Despliegue-practica
   ```

2. Construir las imágenes de Docker:

   ```bash
   docker-compose build
   ```

3. Levantar los contenedores:

   ```bash
   docker-compose up -d
   ```

La aplicación estará disponible en `http://localhost:4200`.

## Endpoints de la API

La API REST del backend expone los siguientes endpoints:

### Personas

- `GET /api/personas`: Obtiene todas las personas.
- `GET /api/personas/{id}`: Obtiene una persona por su ID.
- `POST /api/personas`: Crea una nueva persona.
- `PUT /api/personas/{id}`: Actualiza una persona existente.
- `DELETE /api/personas/{id}`: Elimina una persona.

### Mascotas

- `GET /api/mascotas`: Obtiene todas las mascotas.
- `GET /api/mascotas/{id}`: Obtiene una mascota por su ID.
- `POST /api/mascotas`: Crea una nueva mascota.
- `PUT /api/mascotas/{id}`: Actualiza una mascota existente.
- `DELETE /api/mascotas/{id}`: Elimina una mascota.