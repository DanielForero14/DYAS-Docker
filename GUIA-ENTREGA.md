# Guía de entrega – Taller de Docker

Esta guía te lleva paso a paso hasta la entrega. **Todos los archivos ya están creados**; tu trabajo es ejecutar los comandos, **tomar las capturas** y subirlas.

> Abre una terminal **PowerShell** en la carpeta `DYAS-Docker`. Cada vez que veas 📸, toma una captura (`Win + Shift + S`) y guárdala en `wiki/images/` con el nombre indicado.

## Qué ya está listo

| Carpeta / archivo | Contenido |
|-------------------|-----------|
| `hello-world/` | `Dockerfile` y `hello` (Paso 1). |
| `friendlyhello/` | `app.py`, `requirements.txt`, `.dockerignore`, `Dockerfile`, `docker-compose.yaml`, `docker-compose.traefik.yaml` y `Dockerfile.malorden` (para comparar la caché). |
| `../Wheels-alternative/` | Proyecto del equipo (Parte 4) con su Dockerfile multi-stage. |
| `wiki/` | Las 8 páginas del Wiki ya redactadas; solo faltan capturas y algunos datos `<...>`. |
| `integrantes.txt` | Completa nombres y correos. |
| `.gitignore` | Excluye `data/` y `.env`. |

---

## FASE 0 – Preparación (5 min)

1. Abre **Docker Desktop** y espera a que diga *Engine running*.
2. Verifica:
   ```powershell
   docker version
   docker compose version
   ```
   Debe aparecer una sección **Server**. Si no, Docker Desktop no ha terminado de arrancar.
3. Completa `integrantes.txt` y la tabla de integrantes de `wiki/Home.md`.

---

## FASE 1 – Imágenes propias (Pasos 1 a 3)

```powershell
cd hello-world
docker build --progress=plain -t helloapp:v1 .
docker images helloapp
cd ..
```
📸 `01-hello-build.png` (se ve `hello` impreso y la imagen listada).

```powershell
cd friendlyhello
docker build --check .
```
📸 `02-build-check.png` (debe decir *Check complete, no warnings found*).

```powershell
docker build -t friendlyhello .
docker run --rm -p 4000:8000 friendlyhello
```
Abre http://localhost:4000 → 📸 `03-app-sin-redis.png` (dice *cannot connect to Redis*, es lo esperado).

En **otra** terminal:
```powershell
docker ps
docker exec <CONTAINER_ID> id
```
📸 `04-healthy-usuario.png` (se ve `(healthy)` y `uid=999(app)`). Luego `Ctrl + C` en la primera terminal.

---

## FASE 2 – Caché de capas (Paso 4)

1. En `friendlyhello/app.py` cambia `Hello` por `Hola`.
2. Construye midiendo el tiempo:
   ```powershell
   Measure-Command { docker build --progress=plain -t friendlyhello . | Out-Host }
   ```
   📸 `05-cache-cached.png` (la línea de `pip install` dice **CACHED**). Anota los segundos (`TotalSeconds`).
3. Construye una vez la versión mal ordenada: `docker build -f Dockerfile.malorden -t friendlyhello:malorden .`
4. Cambia `Hola` por `Hello` otra vez en `app.py` y repite:
   ```powershell
   Measure-Command { docker build --progress=plain -f Dockerfile.malorden -t friendlyhello:malorden . | Out-Host }
   ```
   📸 `06-cache-malorden.png` (aquí `pip install` **sí** se ejecuta). Anota los segundos.
5. Escribe ambos tiempos en la tabla de `wiki/Cache-de-capas.md`.

---

## FASE 3 – Docker Compose (Paso 5)

```powershell
docker compose up -d --build
docker compose ps
```
Abre http://localhost:4000 y recarga varias veces → 📸 `07-compose-contador.png` (el número de visitas sube).
📸 `08-compose-ps.png` (salida de `docker compose ps`).

```powershell
docker compose down
```

---

## FASE 4 – Balanceo con Traefik (Paso 6)

```powershell
docker compose -f docker-compose.traefik.yaml up -d --build --scale web=5
docker compose -f docker-compose.traefik.yaml ps
```
📸 `09-traefik-ps.png` (5 réplicas `web` en `healthy`).

Abre **http://localhost:4000** (no `127.0.0.1`) y recarga: el *Hostname* cambia.
📸 `10-hostname-1.png`, `11-hostname-2.png`, `12-hostname-3.png` con 3 hostnames distintos. Escríbelos en `wiki/Balanceo-de-carga.md`.

Abre http://localhost:8080 → 📸 `13-traefik-dashboard.png`.

```powershell
docker compose -f docker-compose.traefik.yaml down
```

---

## FASE 5 – Publicar en Docker Hub (Parte 3)

1. Crea una cuenta en https://hub.docker.com y un repositorio llamado `friendlyhello`.
2. Crea un *Personal Access Token* en Docker Hub (*Account settings → Personal access tokens*) y úsalo como contraseña:
   ```powershell
   docker login
   ```
3. Publica (reemplaza `<usuario>` por tu usuario de Docker Hub):
   ```powershell
   docker build -t <usuario>/friendlyhello:0.1.0 -t <usuario>/friendlyhello:latest .
   docker push <usuario>/friendlyhello:0.1.0
   docker push <usuario>/friendlyhello:latest
   ```
   📸 `14-dockerhub.png` (la página del repositorio con las etiquetas `0.1.0` y `latest`).
4. **Ejercicio 1:** en `docker-compose.yaml` comenta `build: .` y descomenta `image: <usuario>/friendlyhello:0.1.0`. Ejecuta `docker compose up -d` → 📸 `15-ejercicio-1.png`. Luego `docker compose down`.
5. **Ejercicio 2:** cambia la imagen por la de un compañero → 📸 `16-ejercicio-2.png`. Luego `docker compose down`.
6. Vuelve a dejar `build: .` activo y reemplaza `<usuario>` en `wiki/Registro.md`.

---

## FASE 6 – Proyecto del equipo (Parte 4)

El proyecto es **WHEELS** (https://github.com/DanielForero14/Wheels-alternative), con backend en **Node.js**. Está clonado en la carpeta `Taller Doker\Wheels-alternative`, en la rama `corte2`. Ya tiene:

- `backend/Dockerfile` (*multi-stage*, `node:24.22.2-bookworm-slim`, `USER node`, *healthcheck*) y `backend/.dockerignore`.
- `docker-compose.yml` en la raíz, con un volumen para la base de datos SQLite, y `.env.example`.
- `backend/src/server.js`: lee la ruta de la base de datos de `RUTA_BD`.

Desde la carpeta `Wheels-alternative`:

```powershell
Copy-Item .env.example .env
docker compose up -d --build
docker compose ps
```

📸 `17-proyecto-ps.png` (backend `healthy` y `uid=1000(node)`) y 📸 `18-proyecto-app.png` (http://localhost:3000 muestra *API WHEELS funcionando*).

Después: commit y push de estos cambios en el repositorio del proyecto.

---

## FASE 7 – Subir el repositorio

Desde la carpeta `DYAS-Docker`:

```powershell
git add .
git commit -m "Entrega Taller de Docker"
git push
```

Comprueba que `friendlyhello/data/` **no** aparece en GitHub y que el repo es público (o invita al profesor).

---

## FASE 8 – Publicar el Wiki

1. En GitHub, abre el repositorio → pestaña **Wiki** → **Create the first page** → guarda cualquier texto (esto crea el Wiki).
2. En PowerShell, desde la carpeta `Taller Doker` (la que contiene `DYAS-Docker`):
   ```powershell
   git clone https://github.com/DanielForero14/DYAS-Docker.wiki.git
   Copy-Item -Recurse -Force DYAS-Docker\wiki\* DYAS-Docker.wiki\
   cd DYAS-Docker.wiki
   git add .
   git commit -m "Wiki del Taller de Docker"
   git push
   ```
3. Revisa en GitHub que las 8 páginas y las imágenes se ven bien.

---

## Lista final antes de entregar

- [ ] `integrantes.txt` completo.
- [ ] 18 capturas en `wiki/images/` y todos los `<...>` y `__` del Wiki reemplazados.
- [ ] `data/` fuera del repositorio.
- [ ] Imagen en Docker Hub con `0.1.0` y `latest`.
- [ ] Dockerfile multi-stage en el repo del proyecto, app funcionando.
- [ ] Wiki publicado (incluye la Reflexión final; léela y ajústala con tus propias palabras).
- [ ] Enlace del repositorio enviado al profesor.
