# **RESUMEN TEÓRICO – LPIC-2 (201 + 202)**


---

# **MÓDULO 201 – Administración Avanzada de Sistemas Linux**

---

# **1. Planificación de la capacidad**

## **1.1 Medición y análisis del uso de recursos**

La planificación de capacidad se centra en garantizar que un sistema puede soportar su carga actual y futura. Para ello se evalúan varios subsistemas:

### **CPU**

La CPU se analiza mediante:

- **Carga del sistema (load average):** indica cuántos procesos están esperando CPU.  
    Conceptualmente, una carga mayor al número de núcleos es señal de saturación.
    
- **Mecanismos del scheduler:** Linux usa el **CFS (Completely Fair Scheduler)** cuya misión es repartir el tiempo de CPU de forma equitativa según prioridades.
    
- **Estados de los procesos:** ejecución, espera, ininterrumpible (I/O), zombie, etc.
    

### **Memoria**

- Linux utiliza **memoria virtual**, con page cache y mecanismos que priorizan rendimiento.
    
- Una presión de memoria elevada no significa fallo, pero **swap** en exceso sí indica problema.
    
- La gestión dinámica de memoria incluye técnicas como:
    
    - overcommit,
        
    - OOM-Killer,
        
    - page reclaiming.
        

### **Almacenamiento (I/O)**

Se evalúan:

- Latencia,
    
- Throughput,
    
- Número de operaciones por segundo (IOPS),
    
- Colas de espera de disco.
    

Sistemas con alto uso de I/O pueden mostrar retrasos en procesos intensivos en acceso a datos.

### **Red**

El rendimiento de red se evalúa según:

- errores y colisiones,
    
- paquetes descartados,
    
- saturación de enlaces,
    
- congestión en switches o rutas.
    

En entornos productivos, saturaciones pequeñas afectan directamente servicios críticos.

---

## **1.2 Proyección de recursos futuros**

Planificar capacidad futura implica analizar:

- patrones de uso a largo plazo,
    
- crecimiento orgánico de la carga,
    
- necesidades empresariales (picos, nuevas aplicaciones).
    

La teoría incluye:

- modelos predictivos básicos,
    
- identificación de tendencias,
    
- analizar escalabilidad vertical (más CPU/RAM) y horizontal (más nodos),
    
- impacto del almacenamiento en caché,
    
- crecimiento de logs, bases de datos, contenedores y máquinas virtuales.
    

Un buen administrador:

- cuantifica uso diario/semanal,
    
- monitorea con herramientas de series temporales,
    
- determina ventanas de mantenimiento,
    
- evita degradaciones progresivas no detectadas.
    

---

# **2. Kernel de Linux**

## **2.1 Componentes del kernel**

El kernel es un **núcleo monolítico modular**, estructurado en subsistemas:

### **Gestión de procesos**

- El kernel crea, programa y finaliza procesos.
    
- Maneja estados (running, sleeping, zombie, stopped).
    
- Define políticas de prioridad (nice, real-time).
    
- Usa CFS para repartir tiempo CPU.
    

### **Gestión de memoria**

Componentes clave:

- memoria virtual,
    
- paging,
    
- swapping,
    
- VFS cache (page cache),
    
- huge pages,
    
- NUMA (en servidores multiprocesador).
    

El kernel decide qué páginas expulsar y cuándo activar OOM-Killer.

### **Gestión de dispositivos**

Controlado por:

- Drivers incrustados,
    
- Módulos cargables,
    
- udev para asignación dinámica,
    
- /sys para exponer propiedades del hardware.
    

### **Sistemas de archivos**

Linux soporta múltiples FS:

- ext4 (clásico, seguro),
    
- XFS (alto rendimiento),
    
- Btrfs (snapshots, subvolúmenes),
    
- F2FS, tmpfs, NFS, Ceph.
    

La capa VFS (Virtual File System) abstrae interacción con FS concretos.

### **Networking**

El kernel implementa:

- TCP/IP completo,
    
- Netfilter (firewall),
    
- interfaces virtuales (tun/tap),
    
- VLANs,
    
- Bonding/Teaming,
    
- eBPF para filtrado avanzado.
    

### **Seguridad**

- capabilities,
    
- namespaces,
    
- cgroups (limitación de recursos),
    
- LSMs: AppArmor, SELinux,
    
- espacio de usuario restringido (userns).
    

---

## **2.2 Compilación del kernel**

Conceptualmente, compilar un kernel implica:

- Descargar código fuente oficial (vanilla kernel),
    
- Configurar opciones (drivers, FS, arquitectura),
    
- Construir la imagen del kernel + módulos,
    
- Instalar kernel en /boot,
    
- Actualizar GRUB o bootloader equivalente.
    

La teoría importante:

- Diferencias entre kernel genérico y optimizado,
    
- Rol de .config,
    
- Cómo los módulos se vinculan dinámicamente,
    
- Cómo el kernel interactúa con initramfs.
    

---

## **2.3 Gestión en tiempo de ejecución**

El kernel expone parámetros ajustables en tiempo real:

### **Sysctl**

- Permite ajustar configuraciones de red, memoria, seguridad y comportamiento general.
    
- El administrador debe saber qué parámetros afectan rendimiento o seguridad (ej., ip_forward).
    

### **/proc y /sys**

- /proc expone procesos, configuración, estadísticas.
    
- /sys expone información del hardware, buses, dispositivos.
    

### **Módulos del kernel**

Conceptos importantes:

- módulos cargables (.ko),
    
- autoregistro de dependencias,
    
- blacklist,
    
- resolución de problemas cuando un módulo no carga (incompatibilidad, versión).
    

---

# **3. Inicio del sistema**

## **3.1 Secuencia de arranque**

El proceso inicia:

1. **Firmware BIOS/UEFI**: inicialización del hardware.
    
2. **Bootloader (GRUB2)**: carga kernel + initramfs.
    
3. **Kernel**: descubre hardware, monta rootfs inicial.
    
4. **systemd**: inicia servicios según dependencias.
    

El concepto central: systemd estructura el arranque mediante **targets** que reemplazan runlevels clásicos.

## **3.2 Recuperación del sistema**

Incluye:

- modos de rescate (systemd-rescue),
    
- modo emergencia,
    
- manipulación temporal de parámetros del kernel,
    
- reparación de sistemas de archivos,
    
- regenerar initramfs,
    
- reinstalar GRUB,
    
- realizar chroot para recuperar sistemas complejos.
    

## **3.3 Sistemas alternativos de inicio**

Teoría sobre:

- SysV init (basado en scripts),
    
- Upstart (event-driven),
    
- systemd-boot,
    
- LILO (histórico).
    

---

# **4. Sistemas de archivos y dispositivos**

## **4.1 Tipos de FS y características**

### **ext4**

- journaling completo,
    
- timestamps nanosegundos,
    
- sólida estabilidad.
    

### **XFS**

- ideal para grandes volúmenes y paralelismo,
    
- journaling avanzado,
    
- no soporta reducción.
    

### **Btrfs**

- snapshots,
    
- compresión,
    
- subvolúmenes,
    
- checksums.
    

## **4.2 Integridad**

FS modernos usan journaling o copias por escritura para evitar corrupción.  
fsck y herramientas nativas garantizan que errores en apagados no destruyan información.

## **4.3 Opciones de montaje**

Conceptos de seguridad:

- `noexec`: impide ejecutar binarios,
    
- `nosuid`: evita escaladas por suid,
    
- `nodev`: previene creación de dispositivos.
    

---

# **5. Almacenamiento avanzado**

## **5.1 RAID**

Teoría clave:

- RAID 0 → rendimiento
    
- RAID 1 → espejo
    
- RAID 5 → paridad
    
- RAID 6 → paridad doble
    
- RAID 10 → espejo+stripe
    

El examen evalúa:

- redundancia,
    
- reconstrucción,
    
- hot spare,
    
- impacto de fallos en cada nivel.
    

## **5.2 udev y eventos del sistema**

udev gestiona dispositivos dinámicamente mediante reglas.  
Permite automatizar acciones en base a atributos del hardware.

## **5.3 LVM**

Conceptos:

- abstracción del almacenamiento,
    
- pools de espacio (VG),
    
- flexibilidad en la ampliación,
    
- snapshots para pruebas y backups,
    
- migración de datos entre discos sin downtime (pvmove).
    

---

# **6. Redes**

## **6.1 Conceptos fundamentales**

- Interfaces físicas y virtuales,
    
- IP, máscaras, gateway,
    
- rutas estáticas vs dinámicas,
    
- DNS, ARP, ICMP.
    

## **6.2 Redes avanzadas**

Linux permite:

- VLANs,
    
- bridges (puentes L2),
    
- bonding (redundancia/throughput),
    
- tun/tap (VPNs),
    
- policy routing,
    
- túneles GRE/IPIP.
    

## **6.3 Diagnóstico**

Incluye:

- resolución de nombres,
    
- análisis de rutas,
    
- detección de errores de capa 2/3,
    
- problemas de MTU,
    
- conflictos de IP,
    
- congestión.
    

---

# **7. Mantenimiento del sistema**

## **7.1 Compilar software**

Conceptos:

- toolchain,
    
- dependencias de desarrollo,
    
- autotools,
    
- cmake,
    
- fases: configurar → compilar → instalar.
    

## **7.2 Backups**

Teoría:

- tipos de backups,
    
- snapshots vs copias tradicionales,
    
- recuperación granular vs completa,
    
- ciclos de rotación (GFS: abuelo/padre/hijo).
    

## **7.3 Comunicación con usuarios**

Sistemas multiusuario requieren avisar:

- apagados,
    
- mantenimientos,
    
- incidentes.
    

---

# **MÓDULO 202 – Servicios de Red Avanzados**

---

# **1. DNS**

## **1.1 Concepto**

DNS traduce nombres a direcciones.  
Estructura jerárquica: root → TLD → dominios → hosts.

## **1.2 Zonas**

Se definen zonas:

- directa (A/AAAA),
    
- inversa (PTR),
    
- delegaciones,
    
- servidores maestros/esclavos.
    

Registros esenciales:

- SOA (autoridad),
    
- NS (servidores),
    
- MX (correo),
    
- CNAME,
    
- TXT,
    
- SRV.
    

## **1.3 Seguridad**

- DNSSEC garantiza integridad mediante firmas,
    
- TSIG protege transferencias de zona,
    
- evitar open resolvers.
    

---

# **2. Servicios HTTP**

## **2.1 Apache**

Conceptos de:

- VirtualHosts,
    
- módulos,
    
- autenticación,
    
- directory directives,
    
- logging.
    

## **2.2 HTTPS**

Fundamentos:

- criptografía asimétrica,
    
- certificados X.509,
    
- CA,
    
- cadenas de confianza,
    
- renegociación,
    
- confidencialidad + integridad.
    

## **2.3 Squid**

Conceptos:

- proxy directo,
    
- filtrado mediante ACLs,
    
- políticas de caching,
    
- control de acceso por IP o autenticación.
    

## **2.4 Nginx**

Teoría:

- arquitectura basada en eventos,
    
- servidor web ligero,
    
- proxy inverso,
    
- balanceo round-robin,
    
- terminación TLS,
    
- upstreams.
    

---

# **3. Compartición de archivos**

## **3.1 Samba**

Modelo SMB/CIFS:

- compartición de directorios,
    
- autenticación (local o dominio),
    
- control de acceso granular,
    
- roles: standalone, AD member, domain controller (histórico con Samba4).
    

## **3.2 NFS**

Modelo de compartición orientado a UNIX:

- stateless en v3,
    
- stateful en v4,
    
- root_squash,
    
- seguridad con Kerberos (NFSv4 + sec=krb5).
    

---

# **4. Clientes y servidores de autenticación**

## **4.1 DHCP**

Conceptos:

- lease,
    
- asignación dinámica,
    
- reservas,
    
- opciones de DHCP (routers, DNS, dominio),
    
- DHCP relay.
    

## **4.2 PAM**

Modelo modular:

- separación auth/account/password/session,
    
- orden de módulos,
    
- flags de control.
    

## **4.3 LDAP Cliente**

- consultas de directorio,
    
- autenticación centralizada,
    
- esquema y atributos,
    
- integración con PAM y NSS.
    

## **4.4 OpenLDAP Server**

- estructura DIT,
    
- objectClasses,
    
- ACLs,
    
- replicación (syncrepl),
    
- TLS para seguridad.
    

---

# **5. E-Mail**

## **5.1 MTA**

Postfix es modular:

- colas,
    
- transporte,
    
- maps,
    
- políticas de relay,
    
- alias y forwarding.
    

## **5.2 Entrega**

- local,
    
- remota,
    
- transport maps,
    
- filtros.
    

## **5.3 Acceso (MDA)**

Dovecot:

- IMAP vs POP3,
    
- buzones Maildir/mbox,
    
- TLS/SSL.
    

---

# **6. Seguridad del sistema**

## **6.1 Routing y firewall**

- NAT,
    
- forwarding,
    
- firewall de paquetes,
    
- políticas de filtrado.
    

## **6.2 FTP**

- modelo inseguro por defecto,
    
- aislamiento mediante chroot,
    
- diferencias entre vsftpd y ProFTPD.
    

## **6.3 SSH**

- cifrado simétrico + asimétrico,
    
- autenticación por llaves,
    
- túneles,
    
- endurecimiento del servidor.
    

## **6.4 Seguridad general**

- AIDE para integridad,
    
- Fail2ban,
    
- análisis forense básico,
    
- principios de hardening.
    

## **6.5 OpenVPN**

- túneles TLS,
    
- certificados,
    
- control de clientes,
    
- rutas push,
    
- cifrado de tráfico completo.