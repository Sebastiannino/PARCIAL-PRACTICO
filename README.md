# Auditoría de seguridad: Basic Pentesting 1 (VulnHub)

Informe técnico del parcial práctico de **Ética del Hacking**, Unidades Tecnológicas de Santander (UTS).

## Autores

- Juan Sebastián Niño Forero
- Juan Sebastián Pérez

**Fecha del informe:** 23 de noviembre de 2026

## Descripción

Auditoría de seguridad autorizada sobre la máquina virtual **Basic Pentesting: 1** de VulnHub, realizada en un laboratorio aislado montado con VirtualBox. El objetivo del laboratorio es comprometer el sistema de forma remota y obtener privilegios de root.

## Aviso legal y ético

Todo el trabajo se hizo con fines académicos, bajo autorización institucional y dentro de un entorno de laboratorio aislado, en el marco de la Ley 1273 de 2009 de Colombia (delitos informáticos y acceso abusivo a sistemas). Las técnicas descritas **no deben usarse contra sistemas sin autorización expresa**.

## Entorno de laboratorio

| Máquina | Rol | IP | Red |
|---|---|---|---|
| Kali Linux | Atacante | 192.168.56.108 | NAT (solo actualizar herramientas) + Host-Only |
| Basic Pentesting: 1 | Víctima | 192.168.56.107 | Solo Host-Only, sin salida a internet |

Antes de escanear se tomó una instantánea llamada "Lab limpio" de ambas máquinas como punto de restauración.

## Metodología

| Fase | Acción | Herramientas |
|---|---|---|
| 1. Montaje | Construcción del laboratorio aislado | VirtualBox, VBoxManage |
| 2. Descubrimiento | Identificación de hosts activos | nmap -sn |
| 3. Enumeración | Puertos, servicios y directorios web | nmap -sV, gobuster, ftp |
| 4. Análisis de vulnerabilidades | Versiones frente a exploits conocidos | searchsploit |
| 5. Acceso y escalada | Explotación remota y verificación de privilegios | Metasploit Framework |
| 6. Post-explotación | Revisión de evidencias en el sistema | Shell, bash |

## Resultados principales

- Servicios expuestos: FTP (ProFTPD 1.3.3c), SSH (OpenSSH 7.2p2) y HTTP (Apache 2.4.18).
- ProFTPD 1.3.3c contiene un backdoor introducido en el código fuente oficial entre el 28 de noviembre y el 2 de diciembre de 2010 (OSVDB-69562 / BID-45150).
- Con el módulo `exploit/unix/ftp/proftpd_133c_backdoor` de Metasploit se obtuvo ejecución remota de comandos como **root**, sin autenticación.
- El directorio oculto `/secret/` contenía una instalación de WordPress con credenciales de base de datos en texto plano.
- El historial de comandos del sistema evidencia que el backdoor fue instalado de forma deliberada con fines didácticos.

## Hallazgos

| # | Hallazgo | Severidad (CVSS aprox.) |
|---|---|---|
| 1 | Backdoor en ProFTPD 1.3.3c, ejecución remota sin autenticación | Crítica (9.8) |
| 2 | Directorio no enlazado que expone una instalación completa de WordPress | Media (5.3) |
| 3 | Credenciales de base de datos en texto plano en `wp-config.php` | Alta (7.5) |
| 4 | Kernel Linux desactualizado (2017), sin parches | Alta (contextual) |
| 5 | Permisos de `/etc/passwd` alterados (chmod 666) | Alta (7.8) |

## Recomendaciones de mitigación

- Actualizar a una versión soportada de ProFTPD y verificar siempre el checksum del software antes de instalarlo.
- Eliminar instalaciones de prueba antes de pasar a producción y usar autenticación real.
- Guardar credenciales en variables de entorno o en un gestor de secretos.
- Aplicar parches de seguridad de forma periódica.
- Restaurar los permisos estándar de `/etc/passwd` (644) y auditar cambios manuales.

## Conclusión

Una sola mala decisión, instalar software desde una fuente comprometida, bastó para el compromiso total del sistema. La falta de verificación de integridad fue el punto de entrada, y las credenciales expuestas y el directorio de prueba ampliaron el impacto. El laboratorio refuerza la defensa en profundidad, la actualización constante de los sistemas y la validación del origen del software.

## Contenido del repositorio

- `Informe_Tecnico_BasicPentesting1.pdf`: informe completo con las 26 figuras de evidencia.
- `README.md`: este archivo.

## Referencias

- VulnHub. Basic Pentesting: 1. https://www.vulnhub.com/entry/basic-pentesting-1,216/
- Rapid7. ProFTPD-1.3.3c Backdoor Command Execution. Módulo de Metasploit `exploit/unix/ftp/proftpd_133c_backdoor`.
- OSVDB-69562 / BID-45150. ProFTPD Compromised Source Backdoor.
- Congreso de la República de Colombia. Ley 1273 de 2009, delitos informáticos.
