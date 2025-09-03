# Taller 2 - Volúmenes y Bind Mounts en Docker (Linux)

## Autor
**Diego Alejandro Gil Otálora**  
Código: 202222152  
Universidad Pedagógica y Tecnológica de Colombia  
Ingeniería de Sistemas y Computación - Sistemas Distribuidos  
Tunja, 2025  

---

## Ejercicio 1 — Bind mount en modo lectura con Nginx
**Objetivo:** Entender cómo el host controla lo que el contenedor sirve.

```bash
mkdir -p ~/web && echo "<h1>Hola desde bind mount</h1>" > ~/web/index.html
docker run -d --name web-ro -p 8080:80 -v ~/web:/usr/share/nginx/html:ro nginx:alpine
```
<img width="480" height="27" alt="image" src="https://github.com/user-attachments/assets/3aea544c-a534-43d1-87c9-7f881962888a" />
<img width="730" height="273" alt="image" src="https://github.com/user-attachments/assets/914a1bbf-9139-4c39-9ed7-c82ef8f48f80" />


- Acceder a [http://localhost:8080](http://localhost:8080) y verificar.
  
  <img width="493" height="245" alt="image" src="https://github.com/user-attachments/assets/94803e84-fd93-4859-9fbf-9f22a3af7e4b" />

- Editar `index.html` en el host y recargar el navegador para ver cambios.

<img width="841" height="267" alt="image" src="https://github.com/user-attachments/assets/8b3574af-066d-4e0a-9ca2-5bfde18f67be" />

- Probar escribir dentro del contenedor → debe dar error porque el volumen está en solo lectura.

  <img width="739" height="64" alt="image" src="https://github.com/user-attachments/assets/bf7ba625-d372-4b34-a214-21f6f56e30f9" />


---

## Ejercicio 2 — Named volume con PostgreSQL
**Objetivo:** Comprobar persistencia de datos.

```bash
docker volume create pgdata
```
<img width="534" height="78" alt="image" src="https://github.com/user-attachments/assets/15be3d9f-98eb-44f9-ac13-0c555ac71604" />

```bash
docker run -d --name pg -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v pgdata:/var/lib/postgresql/data postgres:16-alpine
```
<img width="746" height="471" alt="image" src="https://github.com/user-attachments/assets/5362d630-917a-4e20-b4de-649293e4113f" />

```bash
docker exec -it pg psql -U postgres -c "CREATE TABLE test(id serial, nombre text);"
docker exec -it pg psql -U postgres -c "INSERT INTO test(nombre) VALUES ('Ada'),('Linus');"
docker exec -it pg psql -U postgres -c "SELECT * FROM test;"
```
<img width="740" height="300" alt="image" src="https://github.com/user-attachments/assets/8a77bd11-4a77-4201-b8af-d0d92aff6d6c" />

```bash
docker rm -f pg
```
<img width="733" height="247" alt="image" src="https://github.com/user-attachments/assets/3cf77ab5-b9af-4d0e-aa6f-c2ab9a0e8100" />


---

## Ejercicio 3 — Volumen compartido entre dos contenedores
**Objetivo:** Producir y consumir datos simultáneamente.

```bash
docker volume create sharedlogs
```
<img width="594" height="74" alt="image" src="https://github.com/user-attachments/assets/17804315-bc62-4ef8-af3b-32e976c6052e" />

```bash
# Productor
docker run -d --name writer -v sharedlogs:/data alpine:3.20 sh -c 'while true; do date >> /data/log.txt; sleep 1; done'
```
<img width="732" height="147" alt="image" src="https://github.com/user-attachments/assets/a05d5e59-9239-4b5d-9025-9030aaa0aba0" />

```bash
# Consumidor
docker run -it --rm --name reader -v sharedlogs:/data alpine:3.20 tail -f /data/log.txt
```
<img width="732" height="309" alt="image" src="https://github.com/user-attachments/assets/6819daf0-728a-48f5-9aa4-6132889abaf0" />

```bash
docker run --rm -v sharedlogs:/data alpine:3.20 sh -lc 'tail -n 3 /data/log.txt'
```
<img width="730" height="72" alt="image" src="https://github.com/user-attachments/assets/cc4cd9a7-abfc-4f60-bf37-37e8509712bb" />


---

## Ejercicio 4 — Backup y restauración de un volumen
**Objetivo:** Aprender a respaldar y restaurar datos.

```bash
# Crear volumen y archivo
docker volume create appdata
docker run --rm -v appdata:/data alpine:3.20 sh -lc 'echo "backup-$(date +%F)" > /data/info.txt'
```

<img width="735" height="60" alt="image" src="https://github.com/user-attachments/assets/35be22b6-b172-4f0e-b8c7-b4e4307dd532" />

```bash
# Backup al host
mkdir -p ~/backups && docker run --rm -v appdata:/data:ro -v ~/backups:/backup alpine:3.20 sh -lc 'cd /data && tar czf /backup/appdata.tar.gz .'
```
<img width="736" height="66" alt="image" src="https://github.com/user-attachments/assets/dcdb43c3-243a-47d2-8420-9b042140274b" />

```bash
# Restaurar en nuevo volumen
docker volume create appdata_restored
docker run --rm -v appdata_restored:/data -v ~/backups:/backup alpine:3.20 sh -lc 'cd /data && tar xzf /backup/appdata.tar.gz'
```

<img width="727" height="72" alt="image" src="https://github.com/user-attachments/assets/87e50a17-797d-4171-ac06-246e7740a58b" />

```bash
# Verificar
docker run --rm -v appdata_restored:/data alpine:3.20 cat /data/info.txt
```
<img width="730" height="60" alt="image" src="https://github.com/user-attachments/assets/a5c4a708-12d8-47f1-9cd3-eed903d36973" />


---

## Reflexión
En los ejercicios aprendí a crear y usar volúmenes en Docker, hacer respaldos y restaurarlos.  
Los principales problemas que tuve fueron errores de sintaxis (`-rm` en vez de `--rm`, `-name` en vez de `--name`, uso de `&` en lugar de `&&`) y archivos inexistentes al usar `tail`.  
Los resolví corrigiendo la sintaxis, creando los archivos o volúmenes antes de usarlos y verificando cada paso con los diferentes comandos de Docker.
