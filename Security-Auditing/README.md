# Auditoría y Análisis de Incidentes con AWS CloudTrail (AWS Lab)

## Descripción general
En este laboratorio se implementa **AWS CloudTrail** para auditar acciones realizadas dentro de una cuenta AWS, con el objetivo de **investigar un incidente de seguridad** que afectó al sitio web del Café.

El laboratorio simula un escenario real donde un atacante modifica reglas de seguridad, accede al sistema operativo y altera archivos del servidor web. A través del análisis de logs, se identifica al responsable y se aplican medidas correctivas.

---

## Arquitectura
La arquitectura del laboratorio incluye:

- **Instancia EC2 – Café Web Server**
- **AWS CloudTrail Trail** con logs almacenados en S3
- **Bucket S3 (monitoring####)** para logs de auditoría
- **Amazon Athena** para análisis mediante SQL
- **Usuarios IAM** (legítimos y maliciosos)
- **Security Groups** del servidor web

<img width="814" height="437" alt="image" src="https://github.com/user-attachments/assets/748a9aaf-dbec-49a1-affc-67d8a0164eec" />

---

## Objetivos
- Configurar un trail de AWS CloudTrail
- Detectar cambios no autorizados en Security Groups
- Analizar logs usando Linux, AWS CLI y Athena
- Identificar al usuario responsable del incidente
- Corregir vulnerabilidades a nivel AWS y sistema operativo

---

## Servicios utilizados
- Amazon EC2
- AWS CloudTrail
- Amazon S3
- Amazon Athena
- AWS IAM
- Amazon VPC (Security Groups)

---

## Implementación del laboratorio

### 1. Observación inicial del sitio web
Se accedió al sitio web del Café y se confirmó que su contenido era legítimo antes del incidente.

<img width="507" height="904" alt="image" src="https://github.com/user-attachments/assets/dbf204ba-d960-4409-ac00-6535dc3561ba" />

---

### 2. Creación del trail de CloudTrail
Se creó un trail llamado `monitor` configurado para registrar eventos de la cuenta y almacenar logs en un bucket S3 dedicado.

<img width="1666" height="304" alt="image" src="https://github.com/user-attachments/assets/1f58f4c2-bef5-4bdd-b140-abe35481629e" />

---

### 3. Detección del sitio web comprometido
Tras la creación del trail, el sitio web fue modificado de forma maliciosa.

<img width="531" height="934" alt="image" src="https://github.com/user-attachments/assets/2238175d-708c-461a-9d20-892365323d83" />

---

### 4. Análisis de Security Groups
Se identificó una regla sospechosa que permitía acceso SSH desde `0.0.0.0/0`.

<img width="1486" height="157" alt="image" src="https://github.com/user-attachments/assets/e0874fe0-88f7-4d6c-9a12-42b7a51b3f61" />

---

### 5. Descarga y análisis de logs con grep
Los logs de CloudTrail fueron descargados desde S3, descomprimidos y analizados con `grep` para identificar eventos relevantes.

<img width="1903" height="460" alt="image" src="https://github.com/user-attachments/assets/c2ab717e-5193-4aca-af98-689414875909" />

<img width="1039" height="995" alt="image" src="https://github.com/user-attachments/assets/82c05d99-b502-484b-8a31-a587279fe0de" />

<img width="988" height="1003" alt="image" src="https://github.com/user-attachments/assets/3883349b-06a4-4844-9858-ee02e031f39c" />

---

### 6. Análisis con AWS CLI (lookup-events)
Se utilizaron comandos `aws cloudtrail lookup-events` para filtrar eventos relacionados con Security Groups.

<img width="1900" height="1006" alt="image" src="https://github.com/user-attachments/assets/d2de02a9-c234-4e11-a2fa-2c1b98497e0b" />

---

### 7. Análisis avanzado con Amazon Athena
Se creó una tabla en Athena para consultar los logs de CloudTrail mediante SQL y descubrir al usuario responsable.

<img width="330" height="516" alt="image" src="https://github.com/user-attachments/assets/c5547a32-7684-449c-b4fe-729e0ca6b8b3" />

<img width="1869" height="822" alt="image" src="https://github.com/user-attachments/assets/21f41d08-84a1-4c15-9cdf-3eb451b73115" />

---

### 8. Identificación del atacante
Mediante consultas SQL se identificó:
- Usuario IAM responsable
- Hora exacta del cambio
- IP de origen
- Método de acceso (CLI o consola)

> 📸 **Captura sugerida:** Resultado final de Athena identificando al atacante.
<details>
  <summary>Ver .csv de la consulta SQL</summary>
#	actor	hack_time_utc	source_ip	eventname	useragent	requestparameters	access_method
1	chaos	2025-12-24T00:02:53Z	44.243.94.245	AuthorizeSecurityGroupIngress	aws-cli/1.18.147 Python/2.7.18 Linux/4.14.355-280.710.amzn2.x86_64 botocore/1.18.6	{"groupId":"sg-0476141b1c602e0df","ipPermissions":{"items":[{"ipProtocol":"tcp","fromPort":22,"toPort":22,"groups":{},"ipRanges":{"items":[{"cidrIp":"0.0.0.0/0"}]},"ipv6Ranges":{},"prefixListIds":{}}]}}	Programmatic (CLI/SDK)
</details>

---

### 9. Remediación a nivel sistema operativo
Se detectó y eliminó un usuario del sistema operativo (`chaos-user`) que había iniciado sesión sin autorización.

<img width="986" height="917" alt="image" src="https://github.com/user-attachments/assets/fb5e6d8a-070a-457f-9ddb-0ff29b119841" />

<img width="534" height="117" alt="image" src="https://github.com/user-attachments/assets/e27da953-f8cd-4d57-8baf-82b2e715a24a" />

---

### 10. Endurecimiento de seguridad SSH
Se deshabilitó la autenticación por contraseña y se corrigieron las reglas del Security Group.

<img width="622" height="90" alt="image" src="https://github.com/user-attachments/assets/c7ec16fc-3fc1-45a8-aea8-16a20780a42c" />

<img width="1650" height="267" alt="image" src="https://github.com/user-attachments/assets/9bc1dbb8-d688-4118-a55c-17b8c948a90b" />

---

### 11. Restauración del sitio web
Se restauró la imagen original del sitio web desde un respaldo local.

<img width="803" height="425" alt="image" src="https://github.com/user-attachments/assets/b807c230-ffdc-42bd-a28e-495a2abd6e89" />

<img width="492" height="872" alt="image" src="https://github.com/user-attachments/assets/4f15e75d-e536-4238-9324-ae015e9a97d9" />

---

### 12. Eliminación del usuario IAM malicioso
El usuario IAM responsable del incidente fue eliminado de la cuenta.

<img width="1607" height="302" alt="image" src="https://github.com/user-attachments/assets/eb3e6854-6a00-4a29-b160-d640327285a4" />

---

## Resultados
- Incidente de seguridad identificado y documentado
- Atacante rastreado mediante logs de auditoría
- Infraestructura y sistema operativo asegurados
- Sitio web restaurado exitosamente

---

## Qué haría distinto en producción
- Habilitar CloudTrail a nivel organización
- Enviar logs a una cuenta centralizada
- Integrar con GuardDuty y Security Hub
- Aplicar IAM con mínimo privilegio estricto
- Automatizar respuestas con Lambda

---

## Conclusión
Este laboratorio demuestra cómo **AWS CloudTrail** es una herramienta clave para **auditoría, investigación forense y respuesta a incidentes**, replicando un escenario real de seguridad en la nube.
