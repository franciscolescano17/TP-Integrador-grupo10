Maximiliano Avendaño
Juan Carlos Toledo
Valentina Ilardo
Francisco Lescano


TP Integrador / Computación aplicada
================================================================================
PASO 1: CONFIGURACIÓN DEL ENTORNO 
================================================================================

QUÉ HACES:
- Descargas una VM Debian desde Blackboard
- La importas en VirtualBox
- Cambias contraseña root a "palermo"
- Cambias hostname a "TPServer"

VERIFICACIÓN:
  hostname → debe devolver "TPServer"
  sudo su - root → contraseña "palermo"

================================================================================
PASO 2: CONFIGURACIÓN DE RED 
================================================================================

QUÉ HACES:
- Configuras IP estática: 192.168.1.41/24
- Configuras Gateway: 192.168.1.1
- Configuras DNS: 8.8.8.8, 8.8.4.4
- TODO en archivo: /etc/network/interfaces

ARCHIVO RESULTANTE:
  /etc/network/interfaces
  - ADDRESS: 192.168.1.41
  - NETMASK: 255.255.255.0
  - GATEWAY: 192.168.1.1
  - DNS: 8.8.8.8, 8.8.4.4

VERIFICACIÓN:
  ip addr show | grep 192.168 → debe mostrar 192.168.1.41/24
  ip route → debe mostrar default via 192.168.1.1

================================================================================
PASO 3: ALMACENAMIENTO 
================================================================================

QUÉ HACES:
1. Agregas un disco de 10GB a la VM (en VirtualBox)

2. Creas 2 particiones:
   - /dev/sdc1: 3GB para /www_dir (donde está el web)
   - /dev/sdc2: 6GB para /backup_dir (donde guardan backups)

3. Montas automáticamente (en /etc/fstab) para que al reiniciar sigan montadas

4. Creas archivo especial /proc/particion con info de particiones

COMANDOS USADOS:
  parted /dev/sdc mklabel msdos
  parted /dev/sdc mkpart primary ext4 0% 30%
  parted /dev/sdc mkpart primary ext4 30% 100%
  mkfs.ext4 /dev/sdc1
  mkfs.ext4 /dev/sdc2
  mount /www_dir
  mount /backup_dir

VERIFICACIÓN:
  lsblk → debe mostrar sdc1 y sdc2
  mount | grep www_dir → debe estar montada
  df -h /www_dir → debe mostrar 3GB
  df -h /backup_dir → debe mostrar 6GB

================================================================================
PASO 4: SERVICIOS 
================================================================================

4.1) SSH (Acceso remoto)
─────────────────────────

QUÉ ES: Servicio que te permite conectarte a la VM desde tu PC

QUÉ HACES:
- Instalas openssh-server
- Lo configuras para acceso CON CLAVE PRIVADA (sin contraseña)
- Usas la clave proporcionada en Blackboard

VERIFICACIÓN:
  systemctl status ssh → active (running)
  Conectas desde Putty con clave privada
  Prompt aparece: root@TPServer:~#

ARCHIVOS MODIFICADOS:
  /etc/ssh/sshd_config

---

4.2) APACHE + PHP (Servidor web)
─────────────────────────────────

QUÉ ES: Servidor web que sirve páginas HTML/PHP

QUÉ HACES:
- Instalas Apache2
- Instalas PHP 7.4 (o superior)
- Instalas módulo PHP para MySQL
- Cambias DocumentRoot de /var/www/html a /www_dir
- Copias archivos: index.php y logo.png a /www_dir

VERIFICACIÓN:
  systemctl status apache2 → active (running)
  php -v → debe mostrar 7.4+
  grep DocumentRoot /etc/apache2/sites-available/000-default.conf → /www_dir
  http://192.168.1.41 en navegador → abre la página web

ARCHIVOS MODIFICADOS:
  /etc/apache2/sites-available/000-default.conf

ARCHIVOS COPIADOS:
  /www_dir/index.php
  /www_dir/logo.png

---

4.3) MARIADB (Base de datos)
─────────────────────────────

QUÉ ES: Sistema de base de datos para guardar información

QUÉ HACES:
- Instalas MariaDB
- Creas base de datos llamada "ingenieria"
- Cargas el archivo db.sql que contiene tablas y datos
- Las tablas son: alumnos, modulos, notas

VERIFICACIÓN:
  systemctl status mariadb → active (running)
  mysql -u root -e "SHOW DATABASES;" → ve "ingenieria"
  mysql -u root ingenieria -e "SHOW TABLES;" → ve alumnos, modulos, notas
  http://192.168.1.41 → la página muestra datos de la BD

ARCHIVOS USADOS:
  /www_dir/db.sql (cargado en MariaDB)

================================================================================
PASO 5: BACKUP AUTOMÁTICO 
================================================================================

QUÉ ES: Script que copia automáticamente archivos importantes

QUÉ HACES:
1. Creas script llamado "backup_full.sh" en /opt/scripts

2. El script backupea 2 cosas:
   - /var/logs → TODOS LOS DÍAS a las 00:00
   - /www_dir → LUNES, MIÉRCOLES, VIERNES a las 23:00

3. Los backups se guardan en /backup_dir

4. Los nombres incluyen fecha: log_bkp_20240302.tar.gz

5. El script acepta argumentos:
   backup_full.sh -help → muestra cómo usarlo
   backup_full.sh /origen /destino → hace backup

6. El script valida que origen y destino existan

7. Programas tareas en crontab para que corran automáticamente

VERIFICACIÓN:
  ls -l /opt/scripts/backup_full.sh → existe y tiene permisos ejecutables
  /opt/scripts/backup_full.sh -help → muestra ayuda
  crontab -l → muestra tareas programadas
  ls /backup_dir → ve archivos .tar.gz con fecha

ARCHIVOS CREADOS:
  /opt/scripts/backup_full.sh

================================================================================
PASO 6: ENTREGABLES 
================================================================================

QUÉ ENTREGAS:

1. REPOSITORIO EN GITHUB
   - README.md con nombres del grupo
   - Comprime en .tar.gz:
     * /root
     * /etc
     * /opt
     * /proc
     * /www_dir
     * /backup_dir
   - /var lo divides en partes pequeñas

2. DIAGRAMA TOPOLÓGICO
   - Dibuja cómo está conectado todo
   - VM → Network → PC
   - Qué servicios corren donde

================================================================================
RESUMEN VISUAL DE LA INFRAESTRUCTURA
================================================================================

TU PC (FÍSICA)
│
├─ Navegador → http://192.168.1.41 → Apache → index.php → MariaDB
│
├─ Putty → SSH (192.168.1.41:22) → Login con clave privada
│
└─ WinSCP → SFTP (192.168.1.41:22) → Copiar archivos

VM DEBIAN (VIRTUAL)
│
├─ SISTEMA OPERATIVO
│  ├─ Hostname: TPServer
│  └─ IP: 192.168.1.41/24
│
├─ ALMACENAMIENTO
│  ├─ /dev/sdc1 (3GB)  → /www_dir
│  │   ├─ index.php
│  │   ├─ logo.png
│  │   └─ db.sql
│  │
│  └─ /dev/sdc2 (6GB)  → /backup_dir
│      ├─ log_bkp_20250114.tar.gz
│      └─ www_bkp_20250114.tar.gz
│
├─ SERVICIOS
│  ├─ SSH (puerto 22)
│  ├─ Apache (puerto 80)
│  ├─ MariaDB (puerto 3306)
│  └─ Cron (tareas automáticas)
│
├─ APLICACIÓN WEB
│  ├─ Apache sirve index.php
│  ├─ index.php conecta a MariaDB
│  └─ Muestra tabla: alumnos, materias, notas
│
└─ SCRIPTS
   └─ /opt/scripts/backup_full.sh
      ├─ Copia /var/logs → /backup_dir
      └─ Copia /www_dir → /backup_dir


