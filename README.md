# Middleware para Integración entre un Ecommerce y un Sistema de Gestión

## Diagrama de arquitectura [Diagrama](https://excalidraw.com/#json=lEjw6OErLDx2qV5T-yX9T,qrIQOvGP390hcqhKix_MUQ)



1. [¿Qué servicios estarían incluidos?](#qué-servicios-estarían-incluidos)
2. [¿Qué componentes agregarías para mejorar la performance?](#qué-componentes-agregarías-para-mejorar-la-performance)
3. [¿Cómo lograrías escalabilidad si aumenta la cantidad de ventas (o la cantidad de ecommerces en el mismo sistema)?](#cómo-lograrías-escalabilidad-si-aumenta-la-cantidad-de-ventas-o-la-cantidad-de-ecommerces-en-el-mismo-sistema)
4. [¿Cómo atenderías la seguridad de la solución?](#cómo-atenderías-la-seguridad-de-la-solución)
5. [¿Qué pros y contras tiene este diseño?](#qué-pros-y-contras-tiene-este-diseño)
6. [¿Qué errores recurrentes podrían existir y cómo se podrían mitigar / atacar?](#qué-errores-recurrentes-podrían-existir-y-cómo-se-podrían-mitigar--atacar)



# ¿Qué servicios estarían incluidos?

* Sincrónico
    * VPC (Virtual Private Cloud)
    * Subnets (privada y publica)
    * NAT Gateway
    * Load Balancer (ELB) 
    * Kubernetes (EKS) 
    * Registry ECR
    * SSM (Systems Manager)
    * Security Groups 
    * IAM 
    * CloudWatch
    * API Gateway (Limite de Tasa 10rqs y 300 req/día)
    * RESTful APIs o Lambda functions
    * Sistema de cache (Redis o Memcache)

* Asincrónico
    * SQS
    * RESTful APIs
    * Limite de cuota diaria al configurar SQS

# ¿Qué componentes agregarías para mejorar la performance?

* Balanceador de carga (ELB)
* Servicio de cache para contenido estático. Elasticache
* Escalabilidad Horizontal: usando docker y EKS (Elastic Kubernetes Service)
    - Escalabilidad automática: usando Auto Scaling Groups
    - Asignando límite de recursos en función de lo que voy a usar CPU y memoria
    - En caso de que un pod esté caído por falla, Kubernetes levanta otro automáticamente.
    - También mejora el deployment de una versión nueva ya que reemplaza la imagen con un tag de versión más nuevo en el caso de latest reemplaza por la subida recientemente.
* Monitorización y Registro: CloudWatch y CloudTrail
* Gestión de Errores y Reintentos: política de Retry con tiempo exponencial backoff 
* Monitorización de Rendimiento

# ¿Cómo lograrías escalabilidad si aumenta la cantidad de ventas (o la cantidad de ecommerces en el mismo sistema)?

* Escalabilidad Horizontal y autoscaling servicio on demand.
* Tener un arquitectura basada en microservicios para bajar el acoplamiento y que se puedan escalar de manera independiente
* Contenedorización (docker)
* Orquestación (Kubernetes)
* Base de datos NoSql para que puedan crecer bajo demanda horizontalmente.
*  Caché (contenido estatico)
* Monitoreo y Ajuste: CloudWatch y CloudTrail

# ¿Cómo atenderías la seguridad de la solución?

* Configuración de subnets públicas y privadas (subnets)
* API Gateway para prevenir contra ataques DDoS e inyección SQL
* Firewall y Security groups con reglas de entrada.
* Dar los permisos adecuados por usuario y por recurso. 
* En caso de EKS se configura 
* Usar HTTPS para encriptar el tráfico y certificados SSL
* Usar IAM para gestionar los permisos de los usuarios dar permisos de manera granular para cada accion
* NAT Gateway para tráfico de salida de la aplicación a internet (actualización de paquetes, etc)
* OAuth 2.0
    * Refresh token
    * Token con vencimiento
    * Scope
    * Grant Type: Authorization Code
* OpenID Connect: autenticar a los usuarios y proporcionar información sobre su identidad

# ¿Qué pros y contras tiene este diseño?

* Pros
    - Seguridad en Profundidad
    - Control de Acceso Granular:
    - Prevención de Ataques
    - Escalabilidad
    - Cifrado y Certificados SSL (Let's Encrypt autorenovados)
    - Monitorización de Seguridad:
    - Mantenimiento automatizado 
    - En la caso de usar Terraform se configura la infraestructura con tres comandos. 
      - terraform init
      - terraform plan
      - terraform apply

* Contras

    - Complejidad
    - Costos
    - Curva de aprendizaje larga

# ¿Qué errores recurrentes podrían existir y cómo se podrían mitigar / atacar?

* Exposición de recursos sensibles:
    - Mitigación: Revisa y ajusta las configuraciones de seguridad de tus recursos en la nube para asegurarte de que solo estén accesibles desde las ubicaciones y servicios necesarios. Utiliza grupos de seguridad y ACL de red para restringir el acceso no autorizado.
    - Ej, si se va a aplicar solución con Kubernetes se necesita un estricto control de roles y permisos IAM y dar a cada usuario la policy especifica que va a usar, por ejemplo como permisos de viewer al conectarse al cluster.
* Actualización
    - Esto se puede solucionar activando la actualización automática de la versión de Kubernetes en AWS.
* Al usar Kubernetes el mismo controller se encarga mantener vivos los pods y de levantarlos en caso de falla
