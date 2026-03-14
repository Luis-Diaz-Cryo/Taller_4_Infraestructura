# Informe de Diagnóstico Técnico de Infraestructura
**Taller 4 – Arquitectura Empresarial**
**Universidad de La Sabana**

---

## 1. Descripción General del Sistema

El sistema analizado es una **plataforma de gestión de propiedades y reservas** desplegada completamente en **AWS Cloud**. La arquitectura sigue un modelo orientado a microservicios con separación clara de capas: frontend, seguridad, API, cómputo, microservicios y datos. También integra plataformas externas como la **API de Airbnb** para sincronización de listados y reservas mediante webhooks.

Los usuarios principales del sistema son: inquilinos (*Tenants*), administradores de propiedades (*Property Managers*), personal de mantenimiento (*Maintenance Staff*) y huéspedes de reserva (*Booking Guests*).

---

## 2. Mapa de Infraestructura – Descripción por Capas

### 2.1 Capa de Frontend
- **CDN** (Content Delivery Network): distribuye contenido estático con baja latencia a nivel geográfico.
- **Web App (React)**: interfaz de usuario servida desde el CDN.

### 2.2 Capa de Seguridad (*Security Layer*)
- **WAF (Web Application Firewall)**: filtra tráfico malicioso antes de que llegue a los servicios internos.
- **Authentication**: gestiona la autenticación de usuarios (presumiblemente mediante JWT o OAuth).

### 2.3 Capa de API (*API Layer*)
- **Load Balancer**: distribuye las solicitudes entrantes entre los servidores de cómputo.
- **Public API & Webhooks**: punto de entrada para integraciones externas, incluyendo eventos de Airbnb.

### 2.4 Capa de Cómputo (*Compute Layer*)
- **API Server 1 y API Server 2**: dos instancias de servidor para procesamiento paralelo de solicitudes. Esto permite escalabilidad horizontal básica.

### 2.5 Capa de Microservicios
- **Integration Service**: gestiona la comunicación con plataformas externas.
- **Booking Service**: maneja la lógica de reservas.
- **Property Service**: administra los datos y operaciones relacionadas con propiedades.

### 2.6 Cola de Eventos (*Event Queue*)
- **Message Queue**: desacopla la comunicación entre microservicios, permitiendo procesamiento asíncrono y resiliencia ante picos de carga.

### 2.7 Capa de Datos (*Data Layer*)
- **Redis Cache**: caché en memoria para reducir la carga sobre la base de datos y mejorar tiempos de respuesta.
- **PostgreSQL Primary**: base de datos relacional principal.
- **PostgreSQL Replica**: réplica de lectura para alta disponibilidad y distribución de carga.
- **S3 Storage**: almacenamiento de objetos para backups y archivos estáticos.
- **Monitoring**: componente de observabilidad conectado a la capa de datos.

### 2.8 Plataformas Externas
- **Airbnb API**: sincroniza listados y reservas con el sistema mediante webhooks y eventos periódicos.

---

## 3. Diagnóstico de Debilidades y Cuellos de Botella

### 3.1 Cuellos de Botella Identificados

| Componente | Problema Potencial | Nivel de Riesgo |
|---|---|---|
| Load Balancer | Punto único de falla si no tiene redundancia | Alto |
| WAF / Authentication | Si están en serie sin redundancia, una caída bloquea todo el acceso | Alto |
| PostgreSQL Primary | Toda la escritura pasa por un solo nodo; bajo carga alta puede ser un cuello de botella | Medio-Alto |
| API Server 1 y 2 | Solo 2 instancias; ante picos de tráfico puede ser insuficiente sin autoescalado | Medio |
| Message Queue | Si la cola no tiene redundancia, los mensajes pueden perderse | Medio |

### 3.2 Puntos Únicos de Falla (SPOF)

- **Load Balancer**: no se evidencia un balanceador secundario ni configuración multi-AZ (múltiples zonas de disponibilidad) explícita en el mapa. Si falla, todo el tráfico queda bloqueado.
- **WAF**: un único WAF sin failover significa que un incidente en este componente puede hacer inaccesible todo el sistema.
- **PostgreSQL Primary**: aunque existe una réplica, la promoción automática de la réplica a primaria no siempre es instantánea, lo que puede generar ventanas de inactividad.

### 3.3 Limitaciones de Escalabilidad

- El **Compute Layer** muestra únicamente dos servidores de API sin indicación de un mecanismo de autoescalado (como AWS Auto Scaling Groups). Ante campañas de alta demanda, estos dos nodos podrían saturarse.
- Los **microservicios** (Integration, Booking, Property) no muestran réplicas ni políticas de escalado independiente, lo que puede limitar el rendimiento de servicios muy demandados (ej. Booking Service durante temporada alta).
- La integración con **Airbnb API** depende de un único flujo de sincronización; si este falla, no se visualiza un mecanismo de reintento robusto o fallback.

### 3.4 Observabilidad y Monitoreo

- El componente de **Monitoring** aparece únicamente conectado a la capa de datos. No se evidencia monitoreo explícito del Compute Layer, los microservicios ni la capa de API, lo que reduce la visibilidad ante fallos en esas capas.
- Se recomienda extender la observabilidad a todos los componentes críticos (Load Balancer, API Servers, microservicios) con métricas de CPU, latencia, tasa de errores y disponibilidad.

### 3.5 Seguridad

- El **WAF** es el primer filtro de seguridad, lo cual es una buena práctica. Sin embargo, no se visualizan controles de seguridad internos entre microservicios (comunicación inter-servicio sin autenticación mutua podría ser un riesgo).
- No se aprecia cifrado explícito en tránsito entre capas internas, aunque podría estar implícito en la configuración de AWS.

---

## 4. Oportunidades de Mejora

### 4.1 Alta Disponibilidad
- Desplegar el Load Balancer y el WAF en **múltiples zonas de disponibilidad (Multi-AZ)** para eliminar puntos únicos de falla.
- Configurar **failover automático** entre PostgreSQL Primary y Replica mediante herramientas como AWS RDS Multi-AZ o Patroni.

### 4.2 Escalabilidad
- Implementar **AWS Auto Scaling Groups** en el Compute Layer para escalar automáticamente el número de API Servers según la carga.
- Evaluar el escalado independiente de cada microservicio mediante **AWS ECS** o **Kubernetes (EKS)**, de manera que el Booking Service pueda escalar de forma autónoma durante picos de reservas.

### 4.3 Observabilidad
- Extender el monitoreo a todas las capas utilizando herramientas como **CloudWatch**, **Prometheus + Grafana** o **Datadog**, cubriendo métricas de latencia, uso de CPU, tasa de errores y disponibilidad de servicios.
- Implementar **alertas proactivas** para detectar degradación de rendimiento antes de que impacte a los usuarios.

### 4.4 Resiliencia en Integración Externa
- Implementar un **mecanismo de reintentos con backoff exponencial** para los webhooks de Airbnb.
- Considerar un patrón **Circuit Breaker** para aislar fallos en la integración externa y evitar que afecten al resto del sistema.

### 4.5 Seguridad Interna
- Adoptar un modelo de **Zero Trust** entre microservicios, con autenticación mutua (mTLS) o tokens de servicio internos.
- Realizar **auditorías de acceso** periódicas sobre la capa de datos y los microservicios.

---

## 5. Conclusión

La arquitectura analizada representa un diseño moderno y bien estructurado sobre AWS Cloud, con separación de responsabilidades por capas, uso de microservicios, caché, cola de mensajes y replicación de base de datos. Estas características son señales positivas de una infraestructura pensada para escalar.

Sin embargo, el diagnóstico revela oportunidades importantes en tres frentes: **eliminar puntos únicos de falla** (Load Balancer, WAF, base de datos primaria), **extender los mecanismos de escalado automático** (Compute Layer y microservicios), y **ampliar la cobertura de observabilidad** más allá de la capa de datos. Abordar estas mejoras permitiría al sistema soportar de manera confiable escenarios de alta demanda y garantizar la continuidad del negocio ante fallos inesperados.

---

## 6. Referencias

- Amazon Web Services. (2023). *AWS Well-Architected Framework*. https://aws.amazon.com/architecture/well-architected/
- Microsoft Azure. (2023). *Cloud Architecture Best Practices*. https://learn.microsoft.com/en-us/azure/architecture/
- Google Cloud. (2023). *Infrastructure Architecture Framework*. https://cloud.google.com/architecture
