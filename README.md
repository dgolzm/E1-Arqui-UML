
# Despliegue de Aplicación Web con AWS

Este proyecto consiste en un backend y frontend desplegados en AWS utilizando múltiples servicios. A continuación, se describe la arquitectura de despliegue y el funcionamiento de cada componente.

## Backend

### 1. **API Gateway**
   El API Gateway se encarga de recibir todas las solicitudes HTTP y redirigirlas hacia los servicios correspondientes. Está configurado con reglas para redirigir las rutas de autenticación (signup y login) a Lambda Function 1 y las otras funcionalidades hacia la app del backend, albergada en Inastancia EC2. Estas últimas rutas son autenticadas con la verificación de tokens JWT (Lambda Function 2).

#### **CORS en API GateWay**
Para garantizar la seguridad de las solicitudes, el API Gateway está configurado con CORS, permitiendo únicamente solicitudes provenientes del frontend.

```
   Access-Control-Allow-Origin: [ https://dsuws8a0krnk6.cloudfront.net, http://localhost:5173 ]
   Access-Control-Allow-Methods: [ GET, POST, PUT, DELETE, * ]
   Access-Control-Allow-Headers: [ authorization, content-type, x-requested-with ]

   ```

### 2. **Lambda Function 1: Auth Service**
   Esta función Lambda maneja las rutas de autenticación, específicamente `signUp` y `login`. Realiza las siguientes funciones:
   - **SignUp**: Registra nuevos usuarios en la base de datos.
   - **Login**: Consulta las credenciales de los usuarios existentes y, si son correctas, genera un token JWT que se devuelve al usuario para su almacenamiento en el frontend.
   
### 3. **Lambda Function 2: Custom Authorizer**
   Esta función Lambda valida los tokens JWT adjuntados a las solicitudes que llegan al API Gateway. Dependiendo de la validez del token, se permite o deniega el acceso a los recursos protegidos. La respuesta de validación se envía de vuelta al API Gateway,  como un IAM policy, el cual lee la API para saber finalmente si redirigir las solicitudes a destino final (Backend App, EC2) o no.

### 4. **Load Balancer NGINX**
   NGINX actúa como balanceador de carga entre el API Gateway y la instancia EC2. Asegura la correcta distribución de las solicitudes que llegan y mejora la escalabilidad de la aplicación.

### 5. **Instancia EC2**
   La instancia EC2 alberga tres contenedores Docker:
   - **Backend App**: Aplicación backend que maneja la lógica de negocio y procesamiento de datos.
   - **MQTT App**: Aplicación que se conecta a un broker MQTT externo, para recibir mensajes relacionados con eventos deportivos compras de bonos y validaciones sobre estas compras.
   - **Base de Datos**: Base de datos que almacena toda la información necesaria para el funcionamiento de la aplicación (usuarios, eventos, datos de las apuestas, etc.).

   Nota: El EC2 recibe las solicitudes del API Gateway solo cuando la validación de tokens es exitosa.


## Frontend

### 1. **CloudFront**
   Amazon CloudFront actúa como red de entrega de contenido (CDN) para servir el frontend de la aplicación. Está conectado a un bucket S3 que contiene la aplicación React del frontend.

### 2. **Bucket S3**
   El bucket S3 alberga el código del frontend de la aplicación. El frontend interactúa con el API Gateway para enviar solicitudes y recibir respuestas, utilizando los tokens JWT generados por el servicio de autenticación. El token JWT se almacena en la consola local, en localStorage.

#### **CORS en Bucket S3**
   Para asegurar que las solicitudes solo provienen del frontend autorizado, se ha configurado CORS en el bucket S3 para aceptar únicamente solicitudes del dominio específico del frontend:
   ```
   [
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "HEAD",
            "GET",
            "PUT",
            "POST",
            "DELETE"
        ],
        "AllowedOrigins": [
            "https://dsuws8a0krnk6.cloudfront.net"
        ],
        "ExposeHeaders": []
    }
]
   ```

## Diagrama de Arquitectura AWS

A continuación se presenta un diagrama que ilustra la arquitectura descrita:

![Diagrama de Arquitectura AWS](/DiagramaArquitecturaAWS.png)


## Pendiente
* ESENCIAL. Configurar NGINX en EC2
* ESENCIAL. Nombre de dominio de primer nivel
* ESENCIAL. Revisar configuración CORS API GateWay
* ESENCIAL. Declarar EndPoints en API GateWay
* ESENCIAL. Asociar un subdominio a API GateWay (Debe asociarse un subdominio a esta (e.g. api.miapp.com))
* ESENCIAL. gestión de tokens JWT con OAuth2 en CustomAuthorizer
* Diagrama UML
