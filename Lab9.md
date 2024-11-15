
# Microservices Architecture Lab 9

## **1. Esquema de Arquitectura de Microservicios**

### **Identificación de Microservicios**
Siguiendo los principios de Domain-Driven Design, se identifican las siguientes áreas funcionales para transformarlas en microservicios independientes:

1. **User Service**:
   - Administra perfiles de usuario, autenticación, autorización, preferencias y sesiones.
   - Dispone de una base de datos específica para la información del usuario.
   - Ofrece APIs para registro, inicio de sesión y gestión de perfiles.

2. **Product Catalog Service**:
   - Responsable de los productos, sus categorías, precios, imágenes e inventarios.
   - Incluye funcionalidades de búsqueda y categorización.
   - Cuenta con una base de datos específica para el catálogo de productos.

3. **Order Service**:
   - Gestiona el carrito de compras, procesamiento de pedidos, pagos y el historial de órdenes.
   - Depende de datos del usuario y del catálogo de productos.
   - Dispone de una base de datos dedicada para transacciones y órdenes.

4. **Customer Support Service**:
   - Gestiona tickets, devoluciones, quejas y retroalimentación.
   - Se integra con el historial de usuarios y órdenes.
   - Usa una base de datos independiente para soporte y tickets.

### **Diagrama de Arquitectura**
El diseño se basa en una arquitectura dirigida por eventos event-driven architecture, donde los microservicios se comunican a través de un message broker para eventos asíncronos. Para comunicaciones críticas, como autenticación, se emplean gRPC.

```
                           +------------------+
      +------------------->|  API Gateway     |
      |                    +------------------+
      |                           |
+----------------+        +----------------+       +-------------------+
|  User Service  |<------>| Order Service  |<----->| Product Catalog   |
+----------------+        +----------------+       +-------------------+
      |                          |                          |
      v                          v                          v
+----------------+        +----------------+       +-------------------+
| Auth Database  |        | Order Database |       | Product Database  |
+----------------+        +----------------+       +-------------------+

                     +---------------------+
                     | Customer Support    |
                     +---------------------+
                     |    Support DB       |
                     +---------------------+

              Message Broker (Event Bus / Kafka)
```

## 2. Plan de Migración 

### **Prioridades de Migración**
1. **User Service**:
   - Primer microservicio a desarrollar, ya que es fundamental para autenticación y autorización.
   - Permite validar usuarios mediante JWT u OAuth2, facilitando la interoperabilidad con otros servicios.

2. **Product Catalog Service**:
   - Segundo en prioridad, ya que sus funcionalidades son esenciales para el catálogo y el proceso de pedidos.

3. **Order Service**:
   - Tercer servicio, ya que depende de la funcionalidad del catálogo y de los usuarios.
   - Incluye lógica para manejar consistencia transaccional (*eventual* o estricta).

4. **Customer Support Service**:
   - Último en ser migrado, dado que tiene una dependencia menor del sistema core y puede funcionar con datos referenciados.

### **Estrategia para Manejar Dependencias de Datos**
- **Base de datos compartida (temporal):** Inicialmente, los servicios consultarán la base monolítica. Gradualmente, cada servicio adoptará su base de datos independiente.
- **Event Sourcing:** Introducir un *message broker* (Kafka o RabbitMQ) para publicar eventos clave y sincronizar datos entre servicios.
- **Scripts de Migración:** Emplear herramientas ETL o scripts específicos para trasladar los datos históricos hacia las nuevas bases de datos.

### **Migración de la Base de Datos**
1. Identificar las tablas y datos relevantes para cada microservicio.
2. Crear esquemas normalizados para cada base de datos según el contexto.
3. Utilizar procesos de *backfilling* para migrar datos históricos.
4. Implementar *dual writes* para que los nuevos servicios escriban en la base monolítica y en la nueva base durante la transición.

---

## **3. Informe de Reflexión**

### **Desafíos Enfrentados**
1. **Consistencia de Datos**:
   - Garantizar consistencia transaccional en múltiples servicios fue un desafío. Se adoptó *eventual consistency* en operaciones no críticas.
2. **Orquestación vs. Coreografía**:
   - Elegir entre un orquestador centralizado o una arquitectura dirigida por eventos. Se optó por un enfoque híbrido.
3. **Dependencias Temporales**:
   - Los primeros servicios migrados dependieron temporalmente de la base monolítica, lo cual requirió una gestión cuidadosa.

### **Decisiones de Diseño**
- Implementar un *API Gateway* para gestionar la comunicación externa.
- Adoptar una arquitectura basada en eventos para escalar operaciones críticas.
- Diseñar bases de datos separadas por contexto para facilitar la escalabilidad y mantenimiento independiente.

### **Conclusión**
La transición a una arquitectura de microservicios mejorará la escalabilidad y flexibilidad del sistema. Este enfoque iterativo mitiga riesgos, garantiza operaciones continuas durante la migración y mejora la calidad del software a largo plazo.