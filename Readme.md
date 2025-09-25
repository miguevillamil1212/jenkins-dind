# 🚀 Jenkins + Docker (DinD) – Despliegue automático de una app web

Este proyecto muestra cómo integrar **Jenkins en Docker** usando **Docker-in-Docker (DinD)** para automatizar el despliegue de una aplicación web sencilla.

El pipeline de Jenkins realiza los siguientes pasos:
1. Limpia el workspace.
2. Clona el repositorio desde GitHub.
3. Construye una imagen Docker de la aplicación web ubicada en `./app`.
4. Crea y ejecuta un contenedor a partir de la imagen construida.
5. Publica el contenedor en el puerto `8081`.
6. Realiza un *smoke test* automático con `curl` para validar el despliegue.

---

## 📂 Estructura del proyecto

.
├─ docker-compose.yml # Orquestación de Jenkins y dependencias
├─ Jenkinsfile # Pipeline declarativo
├─ jenkins/
│ └─ Dockerfile # Imagen personalizada de Jenkins con Docker CLI + Git
└─ app/
├─ Dockerfile # Construcción de la app sobre php:8.2-apache
└─ index.html # Página web de prueba

yaml
Copiar código

---

## ✅ Requisitos previos

- **Docker Desktop** (Windows/Mac) o **Docker Engine** (Linux).
- **Git** instalado.
- En Windows, es recomendable tener habilitado **WSL2**.

---

## ▶️ Pasos para ejecutar el proyecto

### 1) Clonar el repositorio

```bash
git clone https://github.com/<TU_USUARIO>/<TU_REPO>.git
cd <TU_REPO>
2) Levantar Jenkins con Docker Compose
bash
Copiar código
docker compose up -d --build
Esto levantará:

Jenkins en el puerto 8080.

Acceso al socket de Docker para poder ejecutar contenedores.

3) Obtener la contraseña inicial de Jenkins
bash
Copiar código
docker exec -it jenkins-dind bash -lc 'cat /var/jenkins_home/secrets/initialAdminPassword'
Con esa clave:

Ingresa a http://localhost:8080.

Completa la instalación inicial de Jenkins (instalar plugins recomendados).

⚙️ Configuración del Pipeline en Jenkins
En Jenkins → New Item → Pipeline.
Nombre sugerido: Despliegue.

Configuración del pipeline:

Definition: Pipeline script from SCM

SCM: Git

Repository URL: https://github.com/<TU_USUARIO>/<TU_REPO>.git

Branch: */main

Script Path: Jenkinsfile

Desmarcar Lightweight checkout

Guardar y ejecutar el pipeline.

👀 Ver la aplicación desplegada
Accede a http://localhost:8081

Deberías ver la página index.html con el mensaje de despliegue automático.