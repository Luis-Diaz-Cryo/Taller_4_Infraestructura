
# 📄 Informe Técnico del Taller

## 🔖 Nombre del Taller

**Taller – Modelado de Infraestructura de RedExpress**

---

## 👥 Integrantes del equipo

Equipo CyberNexus

---

## 🧠 Descripción general del trabajo

El objetivo del taller fue modelar una **infraestructura tecnológica preliminar para la plataforma logística RedExpress**, identificando los componentes principales del sistema y las posibles áreas críticas que podrían afectar su disponibilidad, rendimiento y escalabilidad.

Durante la actividad se analizó el caso base proporcionado, el cual describe una plataforma utilizada para **gestionar envíos y rastrear paquetes en tiempo real** mediante una aplicación móvil y una plataforma web. A partir de esta información se construyó un **diagrama de infraestructura** que representa la interacción entre usuarios, balanceadores de carga, servicios de API, módulos de procesamiento, base de datos y sistemas de monitoreo.

Adicionalmente, se identificaron **riesgos técnicos importantes**, como latencia en el rastreo, puntos únicos de falla y desafíos de escalabilidad en distintas regiones geográficas.

---

## 🔧 Proceso de desarrollo

El desarrollo del modelo se realizó siguiendo una aproximación gradual.

Primero se identificaron los **actores principales del sistema**, en este caso los usuarios que interactúan con la plataforma a través de la **aplicación móvil y la plataforma web**.

Posteriormente se modeló la **capa de entrada del sistema**, representada por un **balanceador de carga**, encargado de distribuir las solicitudes de los usuarios hacia los servicios internos para evitar sobrecargas en un único servidor.

Después se definió la **capa de servicios**, donde se incluyó un **API Gateway**, encargado de centralizar las solicitudes y dirigirlas hacia los módulos de procesamiento del sistema.

En la siguiente etapa se modelaron los **módulos de procesamiento**, que incluyen:

* procesamiento de rutas
* procesamiento de estados de paquetes

Estos módulos son responsables de manejar la lógica principal del sistema, como el cálculo de rutas logísticas y la actualización del estado de los envíos.

Finalmente se incorporaron componentes de **almacenamiento y monitoreo**, incluyendo una **base de datos centralizada**, un sistema de **recolección de métricas** y un **sistema de alertas**, los cuales permiten supervisar el rendimiento del sistema y detectar posibles fallos.

El diagrama fue realizado utilizando una herramienta de modelado visual (por ejemplo **draw.io**), lo que permitió representar claramente las conexiones entre componentes.

---

## 🧩 Análisis del modelo propuesto

### Estructura del modelo

El modelo está organizado en varias capas principales:

1. **Capa de usuario**

   * Plataforma web
   * Aplicación móvil

2. **Capa de acceso**

   * Balanceador de carga

3. **Capa de servicios**

   * API Gateway

4. **Capa de procesamiento**

   * Procesamiento de rutas
   * Procesamiento de estados de paquetes

5. **Capa de datos**

   * Base de datos

6. **Capa de monitoreo**

   * Recolección de métricas
   * Sistema de alertas

Esta estructura permite separar responsabilidades y facilita la escalabilidad del sistema.

---

### Representación de las necesidades del cliente

El modelo propuesto responde a los requerimientos de RedExpress de varias formas:

* **Alta disponibilidad:**
  El uso de un balanceador de carga permite distribuir las solicitudes de los usuarios y evitar sobrecargas en un único servidor.

* **Procesamiento eficiente:**
  Los módulos especializados permiten manejar diferentes tipos de operaciones del sistema (rutas y estados de paquetes).

* **Monitoreo del sistema:**
  El sistema de métricas y alertas permite detectar problemas rápidamente.

* **Gestión de datos centralizada:**
  La base de datos almacena la información de los envíos y permite mantener consistencia en los registros.

---

### Supuestos realizados

Para el desarrollo del modelo se asumieron algunos aspectos que no estaban completamente definidos en el caso base:

* La base de datos es **centralizada**, aunque podría evolucionar hacia una arquitectura distribuida.
* El API Gateway actúa como **punto principal de comunicación entre clientes y servicios internos**.
* Los módulos de procesamiento pueden escalar horizontalmente si el volumen de solicitudes aumenta.
* El sistema de monitoreo recoge métricas de los módulos principales para detectar fallos o degradación del servicio.

---

## 📈 Diagrama final entregado

Aquí debe insertarse la imagen del modelo desarrollado:

*(Inserte aquí el diagrama de infraestructura de RedExpress)*

---

## 📋 Tabla de actores, entidades o componentes

| Nombre del elemento      | Tipo            | Descripción                                                          | Responsable     |
| ------------------------ | --------------- | -------------------------------------------------------------------- | --------------- |
| Usuarios                 | Actor           | Personas que utilizan la plataforma para rastrear o gestionar envíos | Cliente         |
| Plataforma Web           | Sistema         | Interfaz web para gestionar envíos y rastreo                         | RedExpress      |
| App Móvil                | Sistema         | Aplicación utilizada por usuarios y mensajeros                       | RedExpress      |
| Balanceador de Carga     | Infraestructura | Distribuye el tráfico entre servicios del sistema                    | Infraestructura |
| API Gateway              | Servicio        | Punto de entrada que gestiona las solicitudes del sistema            | Backend         |
| Procesamiento de Rutas   | Servicio        | Calcula rutas logísticas para envíos                                 | Backend         |
| Procesamiento de Estados | Servicio        | Gestiona actualizaciones del estado de paquetes                      | Backend         |
| Base de Datos            | Infraestructura | Almacena información de envíos y estados                             | Infraestructura |
| Recolector de Métricas   | Monitoreo       | Recolecta métricas de rendimiento del sistema                        | DevOps          |
| Sistema de Alertas       | Monitoreo       | Notifica fallos o problemas del sistema                              | DevOps          |

---

## 🔍 Investigación complementaria

### Tema investigado:

**Arquitectura escalable y tolerancia a fallos en plataformas logísticas**

### Resumen

Las plataformas logísticas modernas requieren arquitecturas altamente escalables debido a la gran cantidad de solicitudes que pueden recibir durante eventos de alto tráfico como campañas promocionales o temporadas festivas. Para lograr esto, es común utilizar componentes como **balanceadores de carga**, **microservicios** y **sistemas de monitoreo continuo**.

Una práctica común es evitar **puntos únicos de falla (Single Point of Failure)** mediante redundancia de servicios, replicación de bases de datos y despliegue en múltiples regiones geográficas. Además, los sistemas de monitoreo permiten detectar problemas de rendimiento antes de que impacten a los usuarios finales.

Estas prácticas son relevantes para el caso de RedExpress, ya que ayudan a mitigar riesgos como **latencia en el rastreo en tiempo real**, **fallos en servicios críticos** y **limitaciones en la escalabilidad del sistema**.

---

## 📚 Referencias

* [1] Bass, Len; Clements, Paul; Kazman, Rick. *Software Architecture in Practice*. Addison-Wesley.
* [2] Newman, Sam. *Building Microservices*. O'Reilly Media.
* [3] AWS. *Architecting for High Availability*. [https://aws.amazon.com/architecture/](https://aws.amazon.com/architecture/)

