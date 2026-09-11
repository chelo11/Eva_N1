# Guía de despliegue — Microservicio de Vehículos en AWS EC2

**Proyecto:** cl.matiivilla.vehiculos (ms-vehiculos)
**Entorno de destino:** Amazon EC2 (Ubuntu)
**Fecha:** Septiembre 2026

---

## 1. Objetivo

Este documento describe, paso a paso, el procedimiento completo para compilar el microservicio de vehículos a partir de su código fuente y publicarlo en una instancia EC2 de AWS con Ubuntu, dejándolo configurado como un servicio administrado por `systemd` (inicio automático, reinicio ante fallos y gestión mediante comandos estándar del sistema operativo).

## 2. Datos del entorno

| Recurso | Valor |
|---------|-------|
| **Instancia EC2 principal (EVA_1)** | `54.157.118.185` |
| **Instancia EC2 secundaria (EVA_1_ACTION)** | `34.229.73.22` |
| **Base de datos (AWS RDS)** | `database-1.cj6ww6io2ki7.us-east-1.rds.amazonaws.com:3306` |
| **Base de datos nombre** | `vehiculos` |
| **Usuario BD** | `admin` |
| **Puerto del microservicio** | `9004` |
| **Java requerido** | 21 |

## 3. Requisitos previos

- Archivo fuente del microservicio: código clonado desde el repositorio Git
- Maven Wrapper incluido en el proyecto (archivo `mvnw` / `mvnw.cmd`)
- Cliente SSH: PuTTY, MobaXterm o terminal con SSH
- Datos de conexión a la instancia EC2: IP pública, puerto SSH (22), usuario (`ubuntu`) y llave (`.pem` / `.ppk`)
- Acceso con privilegios `sudo` en la instancia Ubuntu

## 4. Compilación del microservicio

### Paso 1. Clonar el repositorio

```bash
git clone https://github.com/chelo11/Eva_N1.git
cd Eva_N1
```

### Paso 2. Compilar el proyecto

Ejecutar el siguiente comando para compilar y empaquetar el microservicio:

**En Windows:**
```bash
.\mvnw.cmd clean package -DskipTests
```

**En Linux/Mac:**
```bash
chmod +x mvnw
./mvnw clean package -DskipTests
```

> **Nota:** el comando genera la carpeta `target` dentro del proyecto, y dentro de ella el archivo `cl-matiivilla-vehiculos-0.0.1-SNAPSHOT.jar` ejecutable correspondiente al microservicio.

## 5. Aprovisionamiento de la instancia EC2

### Paso 3. Conectarse a la instancia

Conectarse a la instancia mediante SSH utilizando los siguientes datos de conexión:

**Instancia principal (EVA_1):**
```bash
ssh -i "llave.pem" ubuntu@54.157.118.185
```

**Instancia secundaria (EVA_1_ACTION):**
```bash
ssh -i "llave.pem" ubuntu@34.229.73.22
```

> Reemplazar `llave.pem` con la ruta al archivo de llave privada proporcionado.

### Paso 4. Instalar dependencias

Actualizar los repositorios e instalar el entorno de ejecución de Java 21:

```bash
sudo apt update
sudo apt install -y openjdk-21-jre-headless
java -version   # debe mostrar 21.x
```

### Paso 5. Preparar el directorio de despliegue

Crear la carpeta destino para los microservicios y copiar el archivo `.jar` compilado dentro de ella:

```bash
mkdir -p /home/ubuntu/micros
```

Copiar el JAR desde el equipo local a la instancia EC2:

```bash
scp -i "llave.pem" target/cl-matiivilla-vehiculos-0.0.1-SNAPSHOT.jar ubuntu@54.157.118.185:/home/ubuntu/micros/
```

> Repetir el mismo comando para la segunda instancia cambiando la IP a `34.229.73.22`.

## 6. Configuración del servicio systemd

### Paso 6. Crear el archivo de servicio

Crear el archivo de unidad de `systemd` que administrará el microservicio:

```bash
sudo nano /etc/systemd/system/ms-vehiculos.service
```

### Paso 7. Definir el contenido del servicio

Incorporar la siguiente configuración:

```ini
[Unit]
Description=Microservicio MS Vehiculos
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/micros
ExecStart=/usr/bin/java -jar /home/ubuntu/micros/cl-matiivilla-vehiculos-0.0.1-SNAPSHOT.jar
SuccessExitStatus=143
Restart=on-failure
RestartSec=10

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

> **Importante:** el valor de `ExecStart` debe coincidir exactamente con el nombre del archivo `.jar` presente en `/home/ubuntu/micros` (verificar con `ls -la`). Un nombre incorrecto produce el error *"Unable to access jarfile"*.

### Paso 8. Habilitar e iniciar el servicio

Recargar la configuración de `systemd`, habilitar el servicio para que inicie automáticamente con el sistema, y arrancarlo:

```bash
sudo systemctl daemon-reload
sudo systemctl enable ms-vehiculos.service
sudo systemctl start ms-vehiculos.service
```

### Paso 9. Verificar el estado del servicio

```bash
sudo systemctl status ms-vehiculos.service
```

El resultado debe indicar `Active: active (running)`. En caso de error, revisar el detalle en los logs (ver sección 7).

### Paso 10. Habilitar el puerto del microservicio

En la consola de AWS, editar el **Security Group** asociado a la instancia y agregar una regla de entrada (*Inbound rule*):

| Tipo | Protocolo | Puerto | Origen |
|------|-----------|--------|--------|
| Custom TCP | TCP | 9004 | 0.0.0.0/0 (o restringir según corresponda) |

## 7. Verificación del despliegue

Una vez completados los pasos anteriores, verificar que el microservicio responde correctamente:

**Endpoint raíz:**
```bash
curl http://54.157.118.185:9004/
```
Respuesta esperada:
```json
{"mensaje":"Api-vehiculos, Version 1.0"}
```

**Swagger UI (documentación interactiva):**
Abrir en el navegador:
```
http://54.157.118.185:9004/swagger-ui.html
```

**Listar vehículos:**
```bash
curl http://54.157.118.185:9004/api/v1/vehiculos
```

> Repetir las mismas verificaciones con la IP `34.229.73.22` para la segunda instancia.

## 8. Operación y monitoreo del servicio

Comandos de uso frecuente para administrar el ciclo de vida del servicio:

```bash
sudo journalctl -u ms-vehiculos.service -f      # logs en tiempo real
sudo systemctl stop ms-vehiculos.service         # detener el servicio
sudo systemctl restart ms-vehiculos.service      # reiniciar el servicio
sudo systemctl disable ms-vehiculos.service      # deshabilitar inicio automático
```

## 9. Resolución de problemas comunes

**Error: "Unable to access jarfile"**
Indica que la ruta o el nombre del archivo definido en `ExecStart` no coincide con el archivo real. Verificar con `ls -la /home/ubuntu/micros/` y corregir la ruta en el archivo de servicio.

**El servicio reinicia en bucle (auto-restart)**
Revisar el detalle completo del error con `sudo journalctl -u ms-vehiculos.service -n 100 --no-pager`, o ejecutar el `.jar` manualmente (`java -jar archivo.jar`) para visualizar el stack trace completo de la aplicación.

**El microservicio no responde desde fuera de la instancia**
Verificar que el puerto `9004` esté habilitado en el Security Group de la instancia EC2 (ver Paso 10).

**Error de conexión a la base de datos**
Verificar que el Security Group de la instancia RDS permita conexiones entrantes desde las instancias EC2 en el puerto `3306`. También confirmar que la base de datos `vehiculos` existe en la instancia RDS.
