# Deploy — SolucionesIntegrales-Juri

Este README separa claramente la parte de configuración del VPS (tareas de sistema) y la parte de ejecución de los microservicios con Docker, además de las consideraciones y buenas prácticas.

## Índice
- Configuración del VPS (requisitos y tareas de sistema)
- Ejecución de microservicios con Docker (clonar, configurar, levantar)
- Consideraciones y buenas prácticas
- Comandos útiles

---

## Configuración del VPS (si ya tienes Docker instalado y configurado, puedes omitir la mayoría de esta sección)

Objetivo: preparar el servidor para ejecutar contenedores y servicios de forma segura y estable.

Requisitos mínimos del VPS
- Ubuntu 20.04+ o Debian similar
- Acceso root o usuario con sudo
- Dominio apuntado al VPS (A/AAAA records)
- Puertos abiertos: 22 (SSH), 80, 443

Tareas recomendadas (solo si no están hechas)
1. Actualizar paquetes:
```bash
sudo apt update && sudo apt upgrade -y
```
2. Crear usuario de despliegue (opcional):
```bash
sudo adduser deployer
sudo usermod -aG sudo deployer
# configurar claves SSH para deployer
```
3. Firewall (UFW):
```bash
sudo apt install -y ufw
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```
4. Docker (si no está instalado)
- Si ya tienes Docker y está configurado, omite este paso.
- Para instalar Docker (si es necesario):
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```
5. (Opcional) docker-compose:
```bash
sudo apt install -y docker-compose
```

6. Configurar backups y monitoreo (recomendado):
- Configura backups para la base de datos y volúmenes críticos.
- Instala herramientas de monitoreo/logs según tus necesidades.

---

## Ejecución de microservicios con Docker

Esta sección asume que Docker está instalado y funcionando correctamente en el VPS (tal como indicaste).

1) Clonar el repositorio y situarse en la carpeta del proyecto
```bash
git clone https://github.com/SolucionesIntegrales-Juri/deploy.git
cd deploy
```

2) Variables de entorno
- Copia el ejemplo y edita las variables necesarias (.env o archivos de entorno por servicio):
```bash
cp .env.example .env
# editar .env con: DOMINIO, DB_HOST, DB_USER, DB_PASSWORD, etc.
```
- No commitear secretos. Usa secret managers o variables de entorno en el host si es necesario.

3) Construir e iniciar microservicios (docker-compose)
- Si usas docker-compose.yml centralizado:
```bash
docker-compose pull
docker-compose up -d --build
```
- Para forzar reconstrucción sin cache:
```bash
docker-compose build --no-cache
docker-compose up -d
```

4) Comandos útiles para gestión
- Ver estado de servicios:
```bash
docker-compose ps
```
- Ver logs en tiempo real de un servicio:
```bash
docker-compose logs -f <service-name>
```
- Ejecutar comandos dentro de un contenedor:
```bash
docker-compose exec <service-name> bash
# luego ejecutar migraciones, seeders, etc.
```

5) Migraciones y tareas postarranque
- Si alguno de los microservicios requiere migraciones de base de datos o jobs de inicialización, ejecútalos tras levantar los contenedores. Ejemplo:
```bash
docker-compose exec api_service bash
# dentro:
# npm run migrate
# o python manage.py migrate
```

6) Escalado y mantenimiento
- Escalar un servicio:
```bash
docker-compose up -d --scale worker=3
```
- Para actualizar la aplicación:
```bash
git pull origin main
docker-compose pull
docker-compose up -d --build
```

7) Volúmenes y persistencia
- Define volúmenes para bases de datos y datos persistentes en docker-compose.yml para evitar pérdida de datos al recrear contenedores.

---

## Consideraciones y buenas prácticas

Seguridad
- Usar usuarios sin privilegios para ejecutar servicios.
- Deshabilitar login por contraseña en SSH y usar claves públicas.
- Habilitar fail2ban o similar para prevenir ataques por fuerza bruta.

SSL
- Usar Let's Encrypt para HTTPS. Si usas Nginx como proxy inverso, obtén certificados con certbot o usa soluciones basadas en Docker (nginx-proxy + letsencrypt).

Backups
- Programar dumps periódicos de la base de datos (pg_dump, mysqldump) y copia de volúmenes.
- Guardar backups en un almacenamiento externo o S3.

Logging y monitoreo
- Centralizar logs (ELK, Loki, Datadog).
- Monitorizar uso de CPU, memoria y disco.

Actualizaciones y rollback
- Mantener versiones de imágenes con tags; no usar siempre latest en producción.
- Antes de migraciones críticas, crear backups.
- Mantener imágenes anteriores en registry para rollback rápido.

Recursos y límites
- Configurar límites de recursos (memoria/CPU) si es necesario en docker-compose o en el orquestador que uses.

---

## Comandos útiles de troubleshooting
- Ver puertos escuchando:
```bash
ss -tulnp
```
- Ver uso de disco:
```bash
df -h
du -sh /var/lib/docker/*
```
- Reiniciar Docker:
```bash
sudo systemctl restart docker
```

---

Si quieres, puedo:
- Aplicar estos cambios en README.md en la rama update-readme-deploy-Acund01-2025-12-07 (ya creada). 
- Abrir un PR con estos cambios.
- Ajustar la guía con comandos específicos de tus microservicios (nombres de servicios, comandos de migración exactos, puertos) si me das esos detalles.

He actualizado el README separando la configuración del VPS y la ejecución de microservicios con Docker y añadí las consideraciones. Por favor confirma si deseas que haga el commit en la rama indicada y abra el PR.