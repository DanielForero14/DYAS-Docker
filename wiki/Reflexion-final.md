# Reflexión final

## ¿Qué diferencia hay entre una imagen y un contenedor? ¿Y entre un volumen y la capa de escritura del contenedor?

Una **imagen** es una plantilla de **solo lectura** formada por capas (sistema base, dependencias, código). Un **contenedor** es una **instancia en ejecución** de esa imagen: un proceso aislado que comparte el kernel del anfitrión. De una misma imagen salen muchos contenedores, como en el Paso 6, donde la imagen `friendlyhello` generó 5 réplicas.

Cada contenedor agrega encima de la imagen una **capa de escritura** propia. Lo que se escribe ahí **desaparece al eliminar el contenedor** (por ejemplo con `--rm` o `docker compose down`). Un **volumen** vive **fuera** del contenedor: en el Paso 5 el contador de Redis se guardó en `./data` y siguió existiendo después de `docker compose down`. Por eso los datos que deben persistir (bases de datos) van en volúmenes.

## ¿Por qué el orden de las instrucciones del Dockerfile afecta el tiempo de construcción?

Cada instrucción que modifica archivos crea una capa, y Docker reutiliza (cachea) una capa si ni ella ni las anteriores cambiaron. Cuando una capa cambia, **todas las siguientes se reconstruyen**. Si `COPY . .` va antes de `pip install`, cada cambio en `app.py` invalida la instalación de dependencias. Copiando primero `requirements.txt` (o `pom.xml` en Java, o `package.json` y `package-lock.json` en el backend de WHEELS), las dependencias quedan en una capa que solo se rehace cuando cambian las dependencias. En el Paso 4 esto se vio como `CACHED` y en la diferencia de tiempos con `Dockerfile.malorden`.

## ¿Por qué 5 réplicas en una sola máquina no dan alta disponibilidad?

Porque todas comparten el mismo **punto único de falla**: la máquina, su Docker, su red y su disco. Si el equipo se apaga, todas las réplicas caen juntas. Además, Traefik y Redis son una sola instancia cada uno. Las réplicas sí sirven para repartir carga y tolerar la caída de un proceso, pero la alta disponibilidad real requiere distribuir réplicas en **varios nodos** con un orquestador (Kubernetes, Docker Swarm) y replicar también el balanceador y la base de datos.

## ¿Qué riesgos de seguridad tiene una imagen que corre como `root` o que incluye herramientas de compilación?

- **Como `root`:** si un atacante explota una vulnerabilidad de la aplicación, obtiene permisos de administrador dentro del contenedor: puede modificar archivos, instalar herramientas y tiene más posibilidades de escapar al anfitrión (sobre todo si hay volúmenes o `docker.sock` montados). Con `USER app` el daño queda limitado.
- **Con herramientas de compilación** (JDK, Maven, compiladores, gestores de paquetes): la imagen es más grande, tiene **más paquetes con vulnerabilidades** que escanear y parchear, y le da al atacante herramientas para compilar o descargar código malicioso. El *multi-stage* deja en la imagen final solo lo necesario para ejecutar: en Java, el JRE y el `.jar`; en el backend de WHEELS, Node, las dependencias de producción y `src/`, sin las pruebas ni las dependencias de desarrollo.

## Conexión con el proyecto

Lo aprendido se aplicó al backend Node.js de WHEELS: Dockerfile *multi-stage* con `node:24.22.2-bookworm-slim` (versión fija), `npm ci --omit=dev`, usuario `node` sin privilegios y *healthcheck*; `docker-compose.yml` con un volumen para la base de datos SQLite, y la configuración por variables de entorno con `.env.example`. El único cambio al código fue leer la ruta de la base de datos de `RUTA_BD`, y las 148 pruebas del equipo siguieron pasando.

La experiencia del taller también mostró por qué importan estas prácticas en el proyecto: con el orden correcto de capas, cambiar el código del backend no reinstala las dependencias; y con el volumen, los usuarios y viajes no se pierden al recrear el contenedor. Esto deja el backend listo para construirse en el pipeline de CI, escanearse con Trivy y desplegarse igual en cualquier equipo.
