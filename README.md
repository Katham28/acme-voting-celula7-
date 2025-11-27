Proyecto-Célula7
================

Sistema de Votación Serverless para Gadgets – Arquitectura AWS
--------------------------------------------------------------

Este proyecto implementa un sistema completo de **votación serverless**, utilizando AWS para autenticar usuarios, registrar votos, procesar resultados en tiempo real y exponer estadísticas mediante APIs seguras.

La arquitectura está optimizada para **altos picos de tráfico**, **escalabilidad automática**, **costo eficiente** y **seguridad integral**.

---

## Descripción General

El sistema permite que usuarios autenticados voten por un gadget específico. Cada voto se registra una sola vez, se procesa mediante **DynamoDB Streams** y actualiza una tabla agregada con el total de votos por gadget.

Las consultas de resultados se exponen a través de API Gateway de forma segura y escalable.

Este documento resume la arquitectura, componentes, flujos, endpoints y el proceso de despliegue.

---

## Arquitectura General

### **Componentes Principales**

- **Amazon Cognito**
  - Autenticación de usuarios mediante JWT.
  - Autorización para llamadas a API Gateway.

- **Amazon API Gateway**
  - Exposición de endpoints REST (/vote, /results).
  - Validación de tokens Cognito mediante Authorizer.

- **AWS Lambda**
  - `EmitVote`: registra votos.
  - `GetResults`: consulta resultados.
  - `StreamProcessor`: procesa eventos de Streams y actualiza conteos.

- **Amazon DynamoDB**
  - Tabla `Votes`: registro individual por usuario.
  - Tabla `VoteResults`: conteo agregado por gadget.

- **DynamoDB Streams**
  - Notifica la inserción de votos nuevos.
  - Dispara la función `StreamProcessor`.

- **Amazon CloudWatch**
  - Logs de Lambdas.
  - Métricas de API Gateway, DynamoDB y Streams.

- **AWS KMS**
  - Cifrado de datos en DynamoDB y logs.

---

## Estructura del Proyecto

```
/voting-system-c7
├── lambdas/
│   ├── emit_vote/
│   │   └── app.py             # Lambda: Registro de votos
│   ├── get_results/
│   │   └── app.py             # Lambda: Lectura de resultados
│   └── stream_processor/
│       └── app.py             # Lambda: Procesamiento de Streams
├── data/
│   └── sample_votes.json      # Datos de prueba
├── infra/
│   ├── main_stack.yml         # Recursos principales AWS
│   └── iam.yml                # Roles Lambda + políticas DynamoDB
├── scripts/
│   ├── deploy.sh              # Despliegue general
│   └── test_requests.sh       # Pruebas con curl y JWT
└── README.md
```

---

## Despliegue del Sistema

### **Prerrequisitos**

- AWS CLI configurado
- Python 3.9+
- Permisos IAM para crear Lambdas, API Gateway y DynamoDB
- Node o Python para empaquetar Lambdas (según tu implementación)

### **Pasos de despliegue**

1. **Empaquetar Lambdas**
```bash
cd scripts
./package_lambdas.sh
```

2. **Desplegar Roles IAM**
```bash
aws cloudformation deploy \
  --template-file infra/iam.yml \
  --stack-name voting-system-iam \
  --capabilities CAPABILITY_NAMED_IAM
```

3. **Desplegar Infraestructura Principal**
```bash
./deploy.sh
```

4. **Crear Usuario en Cognito**

```bash
USER_POOL_ID=$(aws cloudformation describe-stacks \
  --stack-name voting-system-main \
  --query 'Stacks[0].Outputs[?OutputKey==`UserPoolId`].OutputValue' \
  --output text)

aws cognito-idp admin-create-user \
  --user-pool-id $USER_POOL_ID \
  --username testuser \
  --temporary-password Temp1234! \
  --user-attributes Name=email,Value=test@example.com
```

5. **Probar APIs con JWT**

```bash
./test_requests.sh
```

---

## API Endpoints

### **1. POST /vote**
Registra un nuevo voto por gadget.

**Request**
```json
{
  "gadgetId": "G123"
}
```

**Validaciones**
- Token JWT válido.
- Usuario no ha votado antes.
- gadgetId válido.

---

### **2. GET /results**
Consulta los votos acumulados.

**Query Params**
```
gadgetId: string (opcional)
```

**Ejemplo respuesta**
```json
{
  "gadgetId": "G123",
  "totalVotes": 1520
}
```

---

## Seguridad

- **Autenticación:** Amazon Cognito (JWT)
- **Autorización:** API Gateway Authorizer
- **Cifrado en tránsito:** TLS 1.2+
- **Cifrado en reposo:** AWS KMS para DynamoDB y CloudWatch Logs
- **Validación estricta de inputs en Lambda**
- **Reglas de negocio aplicadas en EmitVote**

---

## Monitoreo

### CloudWatch Logs
- Registros por cada Lambda
- Errores de validación
- Logs de Streams
- Auditoría de votos

### CloudWatch Metrics
- Invocaciones de Lambdas
- Latencia API Gateway
- Consumo de DynamoDB
- Errores 4XX y 5XX

---

## Costos Estimados

Basado en la calculadora de AWS:

- **Lambda:** ~$0.20 por millón de invocaciones  
- **API Gateway:** ~$3.50 por millón de requests  
- **DynamoDB On-Demand:** depende del tráfico, ~low cost  
- **DynamoDB Streams:** costo mínimo por procesamiento  
- **Cognito:** $0.0055 por MAU + autenticación  
- **CloudWatch Logs:** bajo costo por GB almacenado  
- **KMS:** $1 por cada 10,000 solicitudes de cifrado  

---

## Pruebas del Sistema

- Pruebas de votación con JWT válido  
- Pruebas de voto duplicado  
- Pruebas de error por gadget inválido  
- Pruebas de lectura de resultados  
- Validación de conteo acumulado  

---

## Soporte

Para preguntas del proyecto, contactar al equipo de arquitectura académica o revisar la documentación de AWS.

---



