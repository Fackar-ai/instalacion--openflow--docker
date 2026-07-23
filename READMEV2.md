# OpenFlow + OpenCore sobre Docker y Dokploy

![Docker](https://img.shields.io/badge/Docker-27.x-blue)
![Dokploy](https://img.shields.io/badge/Dokploy-Latest-success)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-orange)
![MongoDB](https://img.shields.io/badge/MongoDB-7.x-green)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-3.x-red)
![License](https://img.shields.io/badge/License-MIT-blue)

---

# Tabla de Contenido

- Introducción
- Arquitectura
- Características
- Requisitos Técnicos
- Recursos Recomendados
- Requisitos de Red
- Puertos Utilizados
- Diagrama de Arquitectura
- Preparación del VPS
- Instalación de Docker
- Instalación de Dokploy
- Configuración Inicial de Dokploy
- Configuración DNS
- Despliegue de OpenFlow
- Configuración HTTPS
- Validaciones
- Solución de Problemas
- Backups
- Actualizaciones
- Seguridad

---

# Introducción

Esta guía explica paso a paso cómo instalar **OpenFlow / OpenCore** utilizando Docker y Dokploy sobre un servidor Ubuntu.

El objetivo es disponer de una instalación completamente funcional utilizando:

- Docker
- Docker Compose
- Dokploy
- MongoDB ReplicaSet
- RabbitMQ
- HTTPS automático mediante Traefik
- Let's Encrypt

Al finalizar esta guía tendrás una instalación lista para producción accesible desde un dominio propio.

Ejemplo:

```

https://opencore.midominio.com

```

---

# ¿Qué es OpenFlow?

OpenFlow es una plataforma de automatización empresarial (Workflow Automation Platform) desarrollada por OpenIAP.

Permite crear:

- Automatización de procesos
- Integraciones
- Robots
- Workflows
- Formularios
- APIs
- Gestión documental
- Automatización RPA

La aplicación utiliza MongoDB como base de datos y RabbitMQ como sistema de mensajería.

---

# ¿Qué es Dokploy?

Dokploy es una plataforma Open Source para administrar aplicaciones Docker desde una interfaz web.

Permite:

- Desplegar Docker Compose
- Administrar dominios
- Certificados SSL
- Variables de entorno
- Logs
- Reinicios
- Actualizaciones

Todo desde una interfaz gráfica.

---

# Arquitectura

```

                    INTERNET
                        │
                        │
                    HTTPS :443
                        │
                        ▼
                ┌─────────────────┐
                │     Traefik     │
                │   (Dokploy)     │
                └────────┬────────┘
                         │
                         ▼
                  OpenFlow :3000
                         │
          ┌──────────────┴──────────────┐
          │                             │
          ▼                             ▼
     RabbitMQ                    MongoDB ReplicaSet

```

---

# Componentes

| Componente | Función |
|------------|----------|
| Ubuntu | Sistema Operativo |
| Docker | Contenedores |
| Dokploy | Administración |
| MongoDB | Base de datos |
| RabbitMQ | Cola de Mensajes |
| OpenFlow | Plataforma |
| Traefik | Reverse Proxy |
| Let's Encrypt | Certificados SSL |

---

# Requisitos Técnicos

## Sistema Operativo

Se recomienda:

```

Ubuntu Server 24.04 LTS

```

También funciona con:

- Ubuntu 22.04 LTS

---

## Recursos mínimos

Ambiente de pruebas

| Recurso | Valor |
|----------|-------|
| CPU | 2 Core |
| RAM | 4 GB |
| Disco | 40 GB SSD |

---

## Recursos recomendados

Producción

| Recurso | Valor |
|----------|-------|
| CPU | 4 Core |
| RAM | 8 GB |
| Disco | 100 GB SSD |

---

# Requisitos de Red

Es necesario disponer de:

- IP Pública
- Acceso SSH
- Dominio
- Subdominio

Ejemplo

```

Dominio

empresa.com

Subdominio

opencore.empresa.com

```

---

# Puertos utilizados

| Puerto | Servicio |
|---------|----------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 3000 | OpenFlow |
| 27017 | MongoDB |
| 5672 | RabbitMQ |
| 15672 | RabbitMQ Management |

---

# Requisitos DNS

Antes de iniciar la instalación configure el DNS.

Ejemplo

```

Tipo

A

Host

opencore

Apunta a

130.xxx.xxx.xxx

TTL

Auto

```

Una vez propagado:

```

https://opencore.midominio.com

```

deberá resolver hacia el servidor.

Puede verificar con:

```bash
ping opencore.midominio.com
```

o

```bash
nslookup opencore.midominio.com
```

---

# Preparación del VPS

Actualizar Ubuntu.

```bash
sudo apt update
```

```bash
sudo apt upgrade -y
```

Instalar herramientas.

```bash
sudo apt install curl wget git unzip nano -y
```

Actualizar nuevamente.

```bash
sudo apt autoremove -y
```

Reiniciar el servidor.

```bash
sudo reboot
```

---

# Instalación de Docker

Instalar Docker utilizando el script oficial.

```bash
curl -fsSL https://get.docker.com | sh
```

Agregar el usuario al grupo Docker.

```bash
sudo usermod -aG docker $USER
```

Cerrar la sesión SSH.

Volver a ingresar.

Verificar Docker.

```bash
docker --version
```

Ejemplo

```

Docker version 27.x.x

```

Verificar Docker Compose.

```bash
docker compose version
```

---

# Verificar Docker

Ejecutar:

```bash
docker ps
```

Debe mostrar:

```

CONTAINER ID

```

sin errores.

---

# Instalación de Dokploy

Instalar Dokploy.

```bash
curl -sSL https://dokploy.com/install.sh | sh
```

La instalación puede tardar algunos minutos.

Una vez finalizada aparecerán varios contenedores.

Verificarlos.

```bash
docker ps
```

Debe observar contenedores similares a:

```

dokploy

traefik

postgres

redis

```

---

# Acceso a Dokploy

Abrir el navegador.

```

http://IP_DEL_SERVIDOR:3000

```

Ejemplo

```

http://130.xxx.xxx.xxx:3000

```

Crear el primer usuario administrador.

Este usuario administrará toda la plataforma Dokploy.

---

# Crear un Proyecto

Ingresar al Dashboard.

Seleccionar:

```

Projects

```

Luego

```

Create Project

```

Nombre sugerido.

```

OpenFlow

```

Guardar.

---

# Crear un Servicio Compose

Dentro del proyecto seleccionar:

```

Create Service

```

Luego

```

Docker Compose

```

Asignar un nombre.

```

openflow

```

Guardar.

---

# Clonar el repositorio

Ingresar al servidor mediante SSH.

```bash
cd /opt
```

Clonar el proyecto.

```bash
git clone https://github.com/USUARIO/openflow-instalacion-docker.git
```

Entrar al directorio.

```bash
cd openflow-instalacion-docker
```

Verificar archivos.

```bash
ls
```

Debe existir:

```

docker-compose-dokploy.yml

```

---

# Editar el Docker Compose

Abrir el archivo.

```bash
nano docker-compose-dokploy.yml
```

Modificar el dominio.

Buscar.

```

domain=

```

Cambiar por.

```

domain=opencore.midominio.com

```

Buscar.

```

aes_secret=

```

Generar una clave aleatoria larga.

Ejemplo.

```

BrdYxCCGnWHLTevwW6aYx7ZOK5ALUs1FhnAZX3OBF6ehHLNcP7rKCUCc27gIoLUM

```

Guardar.

```

CTRL + O

ENTER

CTRL + X

```

---

# Subir el Compose a Dokploy

Entrar nuevamente al Dashboard.

Abrir el servicio.

Pegar el contenido completo del archivo.

Guardar.

Presionar:

```

Deploy

```

Esperar que todos los contenedores inicien correctamente.

---

# Despliegue de OpenFlow en Dokploy

Una vez que el archivo `docker-compose-dokploy.yml` ha sido configurado, es momento de desplegar la aplicación utilizando Dokploy.

---

# Crear el Servicio

Desde el Dashboard de Dokploy.

Seleccionar:

```
Projects
```

Abrir el proyecto.

```
OpenFlow
```

Seleccionar.

```
Create Service
```

Elegir.

```
Docker Compose
```

Nombre sugerido.

```
openflow
```

Guardar.

---

# Copiar el Docker Compose

Abrir el archivo.

```bash
cat docker-compose-dokploy.yml
```

Copiar todo su contenido.

Regresar a Dokploy.

Pegar el contenido dentro del editor.

Guardar.

---

# Desplegar

Presionar.

```
Deploy
```

Dokploy descargará automáticamente las imágenes necesarias.

Durante el primer despliegue descargará aproximadamente:

- MongoDB
- RabbitMQ
- OpenFlow

Este proceso puede tardar varios minutos dependiendo de la velocidad de Internet del servidor.

---

# Verificar los Contenedores

Ingresar al servidor.

```bash
docker ps
```

Debe observar algo similar.

```
mongodb

rabbitmq

openflow

mongosetup
```

El contenedor:

```
mongosetup
```

normalmente finalizará con estado.

```
Exited (0)
```

Esto **NO ES UN ERROR**.

Su única función es inicializar el ReplicaSet de MongoDB.

---

# Verificar los Logs

Desde Dokploy.

Seleccionar.

```
Logs
```

o desde la consola.

```bash
docker logs NOMBRE_DEL_CONTENEDOR
```

Ejemplo.

```bash
docker logs openflow-openflow-xxxxxxxx-openflow-1
```

Los siguientes mensajes indican que la aplicación inició correctamente.

```
Connected to mongodb

Connected to rabbitmq

VERSION

Listening

grpc server listening
```

---

# Configuración del Dominio

Una vez desplegado el servicio.

Ingresar a.

```
Domains
```

Seleccionar.

```
Add Domain
```

Configurar.

| Campo | Valor |
|---------|---------|
| Host | opencore.midominio.com |
| Path | / |
| Internal Path | / |
| Container Port | 3000 |
| HTTPS | Activado |
| Strip Path | Desactivado |

Guardar.

---

# Certificado SSL

Si Dokploy ya tiene configurado Let's Encrypt.

Simplemente seleccionar.

```
Let's Encrypt
```

Guardar.

Si la lista aparece vacía.

Primero deberá configurar un proveedor de certificados en la configuración general de Dokploy.

Posteriormente volver al servicio.

Agregar nuevamente el dominio.

---

# Redireccionamiento HTTPS

Una vez generado el certificado.

Abrir.

```
https://opencore.midominio.com
```

No deberá aparecer ninguna advertencia del navegador.

El certificado deberá ser válido.

---

# Verificar el Certificado

Puede verificarlo desde.

https://www.ssllabs.com/ssltest/

Introduzca su dominio.

```
opencore.midominio.com
```

El resultado esperado deberá ser.

```
A

o

A+
```

---

# Validar OpenFlow

Abrir.

```
https://opencore.midominio.com
```

La página principal deberá cargar correctamente.

También es posible acceder directamente a la interfaz.

```
https://opencore.midominio.com/ui/
```

En versiones anteriores también puede funcionar.

```
https://opencore.midominio.com:3001/ui/
```

Sin embargo, en producción se recomienda utilizar únicamente HTTPS mediante Traefik.

---

# Verificar MongoDB ReplicaSet

Ingresar al contenedor.

```bash
docker exec -it openflow-openflow-xxxxxxxx-mongodb-1 mongosh
```

Ejecutar.

```javascript
rs.status()
```

Debe aparecer.

```
PRIMARY
```

Salir.

```javascript
exit
```

---

# Verificar RabbitMQ

Ingresar al navegador.

```
http://IP_SERVIDOR:15672
```

o si el puerto no está publicado.

```bash
docker exec -it openflow-openflow-xxxxxxxx-rabbitmq-1 bash
```

La consola de RabbitMQ deberá estar operativa.

---

# Comandos Útiles

Mostrar todos los contenedores.

```bash
docker ps
```

Mostrar también los detenidos.

```bash
docker ps -a
```

Ver imágenes.

```bash
docker images
```

Ver redes.

```bash
docker network ls
```

Ver volúmenes.

```bash
docker volume ls
```

Ver espacio utilizado.

```bash
docker system df
```

---

# Reiniciar OpenFlow

```bash
docker restart NOMBRE_CONTENEDOR
```

---

# Reiniciar MongoDB

```bash
docker restart mongodb
```

---

# Reiniciar RabbitMQ

```bash
docker restart rabbitmq
```

---

# Reiniciar todos los servicios

```bash
docker compose restart
```

---

# Detener la Aplicación

```bash
docker compose down
```

---

# Iniciar nuevamente

```bash
docker compose up -d
```

---

# Actualizar las Imágenes

Descargar las versiones más recientes.

```bash
docker compose pull
```

Aplicar la actualización.

```bash
docker compose up -d
```

---

# Validaciones Finales

Antes de entregar el servidor en producción valide los siguientes puntos.

| Verificación | Estado Esperado |
|--------------|-----------------|
| Docker funcionando | ✅ |
| Dokploy funcionando | ✅ |
| MongoDB iniciado | ✅ |
| ReplicaSet PRIMARY | ✅ |
| RabbitMQ iniciado | ✅ |
| OpenFlow iniciado | ✅ |
| HTTPS funcionando | ✅ |
| Dominio resolviendo correctamente | ✅ |
| SSL válido | ✅ |
| Acceso a la interfaz | ✅ |

---

# Diagrama Final

```
                        Internet
                            │
                            │
                     HTTPS (443)
                            │
                     +---------------+
                     |    Traefik    |
                     |   Dokploy     |
                     +-------+-------+
                             │
                             │
                     OpenFlow :3000
                             │
          +------------------+------------------+
          │                                     │
          ▼                                     ▼
     RabbitMQ                            MongoDB 7
                                            │
                                            ▼
                                      ReplicaSet rs0
```

---

# Buenas Prácticas

- Utilizar siempre HTTPS.
- Mantener Docker actualizado.
- Cambiar las contraseñas por defecto.
- Generar un `aes_secret` único para cada instalación.
- No modificar el `aes_secret` una vez que el sistema esté en producción.
- Realizar copias de seguridad periódicas de MongoDB.
- Monitorear el consumo de CPU, memoria y disco.
- Mantener actualizado OpenFlow y sus dependencias.

---

# Próximos Pasos

Una vez completada la instalación se recomienda:

1. Crear el primer usuario administrador.
2. Configurar la autenticación.
3. Crear organizaciones y usuarios.
4. Configurar flujos de trabajo.
5. Integrar servicios externos.
6. Implementar una estrategia de respaldos.
7. Configurar monitoreo y alertas.
8. Documentar los cambios realizados en el entorno.
