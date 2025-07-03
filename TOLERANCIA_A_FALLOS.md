# Configuración de Tolerancia a Fallos en Kubernetes

En esta implementación de Kubernetes, se han aplicado diversas estrategias para asegurar la tolerancia a fallos y la alta disponibilidad de los servicios. A continuación, se detallan las configuraciones clave:

## 1. Réplicas de Pods y Autoescalado

Para evitar que un fallo en un único Pod o un aumento en la demanda afecte la disponibilidad del servicio, se utilizan múltiples réplicas y el autoescalado horizontal:

-   **Backend y Frontend:** Ambos `Deployments` inician con **2 réplicas**. Además, se ha configurado un `HorizontalPodAutoscaler` (HPA) para cada uno.
    -   **Backend HPA:** Escala automáticamente entre **2 y 5 réplicas**.
    -   **Frontend HPA:** Escala automáticamente entre **2 y 4 réplicas**.
-   **Mecanismo:** El HPA monitorea el uso de CPU de los Pods. Si el uso promedio supera el **70%**, creará nuevas réplicas para distribuir la carga. Si el uso disminuye, eliminará las réplicas sobrantes hasta alcanzar el mínimo. Esto asegura que la aplicación pueda manejar picos de tráfico sin intervención manual y optimiza el uso de recursos en momentos de baja demanda.

## 2. Liveness Probes (Sondeos de Actividad)

El `Deployment` del **backend** incluye un `livenessProbe`. Este sondeo verifica periódicamente si la aplicación dentro del contenedor sigue activa y saludable.

-   **Configuración:**
    -   **Ruta:** `/actuator/health/liveness`
    -   **Puerto:** `8080`
    -   **Retraso inicial:** **90 segundos**, para dar tiempo a que la aplicación Spring Boot se inicie completamente.
    -   **Intervalo:** Cada 15 segundos.
    -   **Umbral de fallo:** Si la sonda falla 3 veces consecutivas, Kubernetes reiniciará el contenedor automáticamente, asegurando que la aplicación se recupere de bloqueos o estados irrecuperables.

## 3. Readiness Probes (Sondeos de Disponibilidad)

El `Deployment` del **backend** también utiliza un `readinessProbe` para determinar si el contenedor está listo para aceptar tráfico.

-   **Configuración:**
    -   **Ruta:** `/actuator/health/readiness`
    -   **Puerto:** `8080`
    -   **Retraso inicial:** **60 segundos**.
    -   **Intervalo:** Cada 10 segundos.
    -   **Umbral de fallo:** Si la sonda falla 3 veces, el Pod se marcará como "No Listo" y el `Service` dejará de enviarle tráfico hasta que se recupere. Esto evita que los usuarios experimenten errores mientras un Pod se inicia o está sobrecargado.

## 4. Gestión de Recursos (Requests y Limits)

Para garantizar la estabilidad del clúster, todos los `Deployments` (`backend`, `frontend` y `db`) tienen definidos `requests` y `limits` de CPU y memoria.

-   **`requests` (Peticiones):** Es la cantidad de recursos que Kubernetes **garantiza** para el contenedor. Se utiliza para tomar decisiones de scheduling (en qué nodo colocar el Pod).
-   **`limits` (Límites):** Es la cantidad máxima de recursos que un contenedor puede consumir. Evita que un Pod con fugas de memoria o picos de CPU afecte a otros servicios en el mismo nodo (efecto "vecino ruidoso").

Esta configuración previene el agotamiento de recursos en los nodos y hace que el comportamiento de la aplicación sea mucho más predecible.

## 5. Política de Reinicio de Contenedores

Todos los `Deployments` tienen configurada una `restartPolicy` con el valor **`Always`**. Esto le indica a Kubernetes que siempre debe intentar reiniciar un contenedor si este falla o se detiene por cualquier motivo, garantizando que los servicios intenten recuperarse automáticamente.

## 6. Persistencia y Estrategia de la Base de Datos

-   **Persistencia:** La base de datos PostgreSQL utiliza un `PersistentVolumeClaim` (PVC) para almacenar sus datos. Esto asegura que si el Pod de la base de datos falla y es reiniciado, los datos no se perderán.
-   **Estrategia de Despliegue:** El `Deployment` de la base de datos utiliza la estrategia `Recreate`. Esto asegura que, durante una actualización, la instancia antigua se detenga por completo antes de crear la nueva, evitando conflictos de escritura en el volumen persistente.

Estas configuraciones en conjunto crean un sistema robusto, escalable y con una alta capacidad de recuperación ante fallos.