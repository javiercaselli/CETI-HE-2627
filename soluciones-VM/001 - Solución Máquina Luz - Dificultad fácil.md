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
