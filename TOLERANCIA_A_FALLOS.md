# Configuración de Tolerancia a Fallos en Kubernetes

En esta implementación de Kubernetes, se han aplicado diversas estrategias para asegurar la tolerancia a fallos y la alta disponibilidad de los servicios. A continuación, se detallan las configuraciones clave:

## 1. Réplicas de Pods

Para evitar que un fallo en un único Pod afecte la disponibilidad del servicio, se utilizan múltiples réplicas para los componentes críticos:

- **Backend:** El `Deployment` del backend está configurado para correr **2 réplicas**. Esto significa que siempre habrá dos instancias del backend en ejecución. Si una de ellas falla, el `Service` de Kubernetes redirigirá el tráfico a la réplica saludable, manteniendo el servicio disponible.

- **Frontend:** Al igual que el backend, el `Deployment` del frontend también está configurado con **2 réplicas**, garantizando que la interfaz de usuario siga siendo accesible aunque uno de los Pods deje de funcionar.

## 2. Liveness Probes (Sondeos de Actividad)

El `Deployment` del **backend** incluye un `livenessProbe`. Este sondeo verifica periódicamente si la aplicación dentro del contenedor sigue activa y saludable.

- **Configuración:**
  - **Ruta:** `/actuator/health/liveness`
  - **Puerto:** `8080`
  - **Intervalo:** Cada 15 segundos.
  - **Umbral de fallo:** Si la sonda falla 3 veces consecutivas, Kubernetes considerará que el contenedor no está saludable y lo reiniciará automáticamente.

Esta configuración asegura que si la aplicación se bloquea o entra en un estado irrecuperable, se reinicie de forma automática para restaurar su funcionalidad.

## 3. Readiness Probes (Sondeos de Disponibilidad)

El `Deployment` del **backend** también utiliza un `readinessProbe`. Este sondeo determina si el contenedor está listo para empezar a aceptar tráfico.

- **Configuración:**
  - **Ruta:** `/actuator/health/readiness`
  - **Puerto:** `8080`
  - **Intervalo:** Cada 10 segundos.
  - **Umbral de fallo:** Si la sonda falla 3 veces, el Pod se marcará como "No Listo" y el `Service` dejará de enviarle tráfico hasta que vuelva a estar disponible.

Esto es útil para evitar que se envíe tráfico a un Pod que se está iniciando o que está temporalmente sobrecargado y no puede procesar nuevas solicitudes.

## 4. Política de Reinicio de Contenedores

Todos los `Deployments` (`backend`, `frontend` y `db`) tienen configurada una `restartPolicy` con el valor **`Always`**.

- **Efecto:** Esta política le indica a Kubernetes que siempre debe intentar reiniciar un contenedor si este falla o se detiene por cualquier motivo. Esto garantiza que los servicios intenten recuperarse automáticamente de fallos inesperados.

## 5. Persistencia de Datos para la Base de Datos

Para la base de datos PostgreSQL, la persistencia de los datos se gestiona a través de un `PersistentVolumeClaim` (PVC) llamado `pgdata`.

- **Configuración:**
  - **`accessModes`:** `ReadWriteOnce`, lo que significa que el volumen solo puede ser montado por un único nodo a la vez.
  - **`resources`:** Solicita `100Mi` de almacenamiento.

El `Deployment` de la base de datos (`db`) utiliza este PVC para montar el volumen en la ruta `/var/lib/postgresql/data`. Esto asegura que aunque el Pod de la base de datos falle y sea reiniciado, los datos no se perderán, ya que están almacenados en un volumen persistente fuera del ciclo de vida del Pod.

## 6. Estrategia de Despliegue de la Base de Datos

El `Deployment` de la base de datos (`db`) utiliza una estrategia de despliegue (`strategy`) de tipo **`Recreate`**.

- **Efecto:** Cuando se realiza una actualización, Kubernetes primero terminará el Pod existente de la base de datos antes de crear uno nuevo con la nueva configuración. Aunque esto implica un breve tiempo de inactividad durante la actualización, es una estrategia segura para bases de datos de instancia única, ya que previene que dos instancias intenten escribir en el mismo volumen `ReadWriteOnce` simultáneamente, lo que podría causar corrupción de datos.

Estas configuraciones en conjunto crean un sistema robusto y con una alta capacidad de recuperación ante fallos.
