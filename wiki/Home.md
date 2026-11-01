# Taller de Docker

**Curso:** Diseño y Arquitectura de Software – Universidad de La Sabana – 2026

## Integrantes

| Nombre | Correo institucional |
|--------|----------------------|
| Daniel Felipe Forero Sánchez | danielforsa@unisabana.edu.co |

## Propósito del taller

Aprender a **crear imágenes propias**, **ejecutar contenedores** y **componer aplicaciones de varios servicios** con Docker y Docker Compose, aplicando buenas prácticas (versiones fijas, orden de capas, `.dockerignore`, usuario sin privilegios, *healthcheck* y *multi-stage*), escalando un servicio detrás de **Traefik**, publicando la imagen en **Docker Hub** y contenerizando la aplicación **Java / Spring Boot** del proyecto del equipo.

## Contenido

1. [Imágenes](Imagenes)
2. [Caché de capas](Cache-de-capas)
3. [Compose](Compose)
4. [Balanceo de carga](Balanceo-de-carga)
5. [Registro](Registro)
6. [Proyecto](Proyecto)
7. [Reflexión final](Reflexion-final)

## Repositorio

- Carpeta `hello-world/`: Paso 1.
- Carpeta `friendlyhello/`: Pasos 2 a 6 (aplicación Flask + Redis + Traefik).
- Parte 4: el Dockerfile multi-stage está en el repositorio del proyecto, [Wheels-alternative](https://github.com/DanielForero14/Wheels-alternative) (backend en Node.js).
