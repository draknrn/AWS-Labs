# Working with Amazon S3 – Uso compartido seguro de archivos y notificaciones por eventos (AWS Lab)

## Descripción general
En este laboratorio creé y configuré un **bucket de Amazon S3** para compartir archivos de forma segura con un **usuario externo** perteneciente a una empresa de medios.  
Además, configuré **notificaciones por eventos** para que cualquier cambio en el contenido del bucket genere automáticamente una **alerta por correo electrónico** utilizando **Amazon SNS**.

Este laboratorio simula un escenario real donde usuarios de terceros necesitan **acceso restringido** para subir y gestionar archivos, mientras los administradores mantienen **control, seguridad y visibilidad**.

---

## Arquitectura
La arquitectura de la solución está compuesta por:

- Un **bucket de Amazon S3** utilizado como repositorio compartido de imágenes
- Un **grupo y usuario IAM (mediacouser)** que representa a una empresa externa de medios
- **Políticas IAM de grano fino** que restringen el acceso a un prefijo específico de S3
- Un **tópico de Amazon SNS** para notificaciones por eventos
- Suscripciones por correo electrónico para recibir alertas cuando se crean o eliminan objetos

<img width="1068" height="724" alt="image" src="https://github.com/user-attachments/assets/41fcbc34-7dd3-4318-91fd-c2a9629b5ce1" />

---

## Objetivos
- Crear y configurar un bucket S3 usando AWS CLI
- Otorgar acceso de escritura controlado a un usuario IAM externo
- Validar permisos IAM mediante operaciones reales
- Configurar notificaciones por eventos en S3
- Enviar alertas por correo electrónico usando Amazon SNS

---

## Servicios y tecnologías utilizados
- Amazon S3
- AWS Identity and Access Management (IAM)
- Amazon SNS
- AWS CLI (`s3` y `s3api`)
- Amazon EC2 (CLI Host)
- JSON (políticas IAM y configuración de notificaciones)

---

## Desarrollo del laboratorio

### 1. Conexión al CLI Host y configuración de AWS CLI
Me conecté a la instancia **CLI Host EC2** usando EC2 Instance Connect y configuré AWS CLI con las credenciales proporcionadas.

Para validar que AWS CLI estaba correctamente configurado, se ejecutó el siguiente comando:

```bash
aws sts get-caller-identity
```
<img width="904" height="108" alt="image" src="https://github.com/user-attachments/assets/995316d3-27d5-428f-80c6-0beddbbafbb1" />

---

### 2. Creación e inicialización del bucket S3
Usando AWS CLI, creé un bucket S3 con un nombre único y cargué imágenes iniciales bajo el prefijo `images/`.

```bash
aws s3 mb s3://cafe-pincheira --region us-west-2
aws s3 sync ~/initial-images/ s3://cafe-pincheira/images
aws s3 ls s3://cafe-pincheira/images/ --human-readable --summarize
```

<img width="925" height="104" alt="image" src="https://github.com/user-attachments/assets/da1ef0b7-f633-4a30-a01c-2f9dbd61ca14" />

<img width="863" height="116" alt="image" src="https://github.com/user-attachments/assets/2d5b7158-29e7-4652-9380-893ee046bec3" />

---

### 3. Revisión de permisos del grupo y usuario IAM
Revisé el **grupo IAM mediaco** y su política asociada, la cual otorga permisos a nivel de objeto únicamente dentro del prefijo `images/`, aplicando el principio de **mínimo privilegio**.

```bash
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": [
				"s3:ListAllMyBuckets",
				"s3:GetBucketLocation"
			],
			"Resource": [
				"arn:aws:s3:::*"
			],
			"Effect": "Allow",
			"Sid": "AllowGroupToSeeBucketListInTheConsole"
		},
		{
			"Action": [
				"s3:ListBucket"
			],
			"Resource": [
				"arn:aws:s3:::cafe-*",
				"arn:aws:s3:::cafe-*/*"
			],
			"Effect": "Allow",
			"Sid": "AllowRootLevelListingOfTheBucket"
		},
		{
			"Action": [
				"s3:PutObject",
				"s3:GetObject",
				"s3:GetObjectVersion",
				"s3:DeleteObject",
				"s3:DeleteObjectVersion"
			],
			"Resource": "arn:aws:s3:::cafe-*/images/*",
			"Effect": "Allow",
			"Sid": "AllowUserSpecificActionsOnlyInTheSpecificPrefix"
		}
	]
}
```
---

### 4. Pruebas de acceso del usuario externo (mediacouser)
Validé las acciones permitidas (subir, visualizar y eliminar archivos) y confirmé que las acciones no autorizadas, como modificar permisos del bucket, fueron correctamente bloqueadas.

<img width="1772" height="656" alt="image" src="https://github.com/user-attachments/assets/e6b1ef01-d5af-41e6-85d1-c1db214fddf1" />

<img width="1773" height="762" alt="image" src="https://github.com/user-attachments/assets/56d463c0-be7d-49b4-9709-84d95b58c70b" />

<img width="817" height="760" alt="image" src="https://github.com/user-attachments/assets/a3ae9995-405f-409e-9ccf-c7ded6d4707d" />

---

## Notificaciones por eventos con Amazon SNS

### 5. Creación y configuración del tópico SNS
Se creó un tópico SNS llamado `s3NotificationTopic` y se confirmó una suscripción por correo electrónico para recibir notificaciones.

<img width="1914" height="582" alt="image" src="https://github.com/user-attachments/assets/de186382-22e5-44db-90bc-d45a61d65f2c" />

---

### 6. Configuración de notificaciones por eventos en S3
Se aplicó una configuración de notificaciones para que Amazon S3 publique eventos cuando se crean o eliminan objetos bajo el prefijo `images/`.

<img width="1458" height="375" alt="image" src="https://github.com/user-attachments/assets/d1346e0e-2a60-4deb-9f95-92d3b96b3a2b" />

---

### 7. Pruebas de notificaciones por eventos
Las acciones de carga y eliminación de objetos generaron notificaciones por correo electrónico, mientras que las operaciones de solo lectura no, confirmando el correcto funcionamiento de la configuración.
<img width="654" height="691" alt="image" src="https://github.com/user-attachments/assets/9391dc2e-23ca-43c3-b392-f7927bff7131" />

<img width="658" height="671" alt="image" src="https://github.com/user-attachments/assets/9b2985e8-0bb5-4b6e-95e6-53f8e550dc7e" />

#### Comprobación de credenciales
<img width="509" height="108" alt="image" src="https://github.com/user-attachments/assets/bc047f3a-ff61-4b2f-abe2-1b4d96948f2a" />

#### Acciones PUT & GET
<img width="1323" height="258" alt="image" src="https://github.com/user-attachments/assets/e04cbcb3-946d-4e8e-98c4-298ab4c6e565" />

#### Acción DELETE
<img width="978" height="174" alt="image" src="https://github.com/user-attachments/assets/e2c54f09-bf9f-4da9-82fc-94d73d2aaa5d" />

<img width="861" height="73" alt="image" src="https://github.com/user-attachments/assets/71efa94a-e67e-4b54-a823-29846cb3059a" />

#### Acción cambiar permisos de visibilidad
<img width="1902" height="72" alt="image" src="https://github.com/user-attachments/assets/38111691-02d0-42ac-8db2-843e9ad3f9fd" />

---

## Resultados
- Bucket S3 configurado de forma segura para colaboración externa
- Permisos IAM correctamente restringidos por prefijo
- Acciones no autorizadas bloqueadas correctamente
- Notificaciones por correo en tiempo real ante cambios en los archivos

---

## Aprendizajes clave
- Las políticas IAM basadas en prefijos son esenciales para compartir S3 de forma segura
- Los usuarios externos no deben tener permisos a nivel de bucket
- Las notificaciones por eventos en S3 permiten un mecanismo simple de auditoría
- Amazon SNS es eficaz para alertas operacionales en tiempo real

---

## Qué haría distinto en un entorno de producción
- Utilizar **Infraestructura como Código** (CloudFormation o Terraform)
- Reemplazar usuarios IAM por **acceso federado y roles IAM**
- Habilitar **S3 Server Access Logging o eventos de datos en CloudTrail**
- Cifrar objetos usando **SSE-KMS**
- Integrar notificaciones con herramientas centralizadas de monitoreo

---

## Conclusión
Este laboratorio demuestra experiencia práctica en **uso compartido seguro de archivos con Amazon S3**, **control de acceso mediante IAM** y **notificaciones basadas en eventos usando Amazon SNS**, alineado con buenas prácticas reales de seguridad y operación en la nube.
