# Seguimiento

[← Volver al README](../README.md)

## Preguntas pendientes

Para los docentes y el centro:

- [ ] **ASGBD, urgente:** ¿qué edición de Oracle y qué licencia usaremos en las sedes? La réplica con promoción (Data Guard) solo está en la edición Enterprise. ¿Funciona esa versión en Windows Server 2025?
- [ ] **SAD:** ¿vale el nivel 1 de seguridad en las sedes, con las mejoras documentadas, o se espera un cortafuegos dedicado por sede? ¿Qué protección frente a denegación de servicio se espera?
- [ ] **Centro (Proxmox):** ¿tendremos usuario propio con cuotas de RAM, CPU y disco? ¿Podemos crear redes virtuales propias? ¿Se puede abrir el puerto UDP 51820 desde fuera para la VPN? ¿Hay copias o instantáneas de las máquinas? (El docente está de acuerdo; previsto en un par de semanas.)
- [ ] **Centro (portátiles):** ¿pueden prestarnos un portátil para el técnico de la demostración y, si hace falta, otro para la sede B? El de la sede B necesitaría permisos de administrador, la virtualización activada y poder desactivar la VBS.
- [ ] **Centro (licencias):** ¿tiene el centro Azure Dev Tools for Teaching u otra vía para licencias educativas de Windows Server y Windows 11?
- [ ] **ASO:** ¿un único dominio con un controlador por sede o un dominio por sede? Afecta a las cuentas y a cómo se comporta la caída de una sede.
- [ ] **IPE II:** ¿qué se espera de la parte de empresa en la defensa?

Para el grupo:

- [ ] Comprobar el PC de la sede B (casi seguro, el de un compañero): RAM, CPU, virtualización activada, edición de Windows (la Home no acepta escritorio remoto entrante) y que se pueda desactivar la VBS.
- [ ] Comprobar si hay CGNAT en la línea de la sede B y si el router permite DNS dinámico y abrir puertos.
- [ ] Decidir el motivo de la caída de la sede A (el gato o cualquier otra cosa que no sea Windows Update).
- [ ] Decidir si se demuestra la restauración de una copia de seguridad y cómo.
- [ ] Elaborar la tabla de direcciones IP: equipos reales remotos y máquinas virtuales.
- [ ] Buscar nombres para nuestra empresa y para la empresa cliente. Bloquean el dominio de AD, los nombres DNS y los certificados.
- [ ] Conseguir las licencias educativas antes de instalar AD en las sedes.
- [ ] Medir el consumo de los servidores (y monitor y router...) de las sedes y nuestro servidor propio para dimensionar los SAI.
- [x] Regenerar `diagrama-red.png` con los datos actualizados (fuente editable en `docs/diagramas/diagrama-red.html`).
- [x] Montar OpenProject de prueba (MV de Ubuntu Server en el PC de Jordi).
- [x] VPN WireGuard para acceder a OpenProject desde fuera.

## Contenidos por ver y dónde encajarían

El proyecto está pensado como un esqueleto al que se le enganchan piezas: lo caro de cambiar son los cimientos (dominio, motor de base de datos, VPN y direcciones). Lo que vaya saliendo en clase será, casi siempre, un añadido pequeño o una sección más de la memoria. No hace falta meterlo todo: basta con que cada módulo tenga su parte visible.

| Módulo | Lo que suele verse | ¿Lo cubre el proyecto? | Si no, cómo encajaría |
| --- | --- | --- | --- |
| ASO | Dominios, AD, GPO, LDAP, integración Windows-Linux, scripts, servicios y monitorización | Casi todo | Scripts de PowerShell o Bash (altas masivas de usuarios, copias); quizá OpenLDAP o Samba, documentado |
| SRI | DNS, DHCP, servidores web, FTP, correo, proxy, acceso remoto, NAT y VPN | Casi todo, menos el correo | Un servidor de correo en Docker en nuestra empresa, que además serviría para los avisos de OpenProject |
| IAW | Servidores web, CMS, PHP, despliegue y contenedores | Sí | Si piden un CMS, el blog de nuestra empresa pasa a ser un WordPress en Docker |
| ASGBD | Instalación, usuarios, permisos, auditoría, copias y alta disponibilidad | De sobra | Oracle con Data Guard en las sedes y Oracle Free en nuestra empresa |
| SAD | Seguridad física, copias, criptografía, cortafuegos, proxy, VPN, IDS, alta disponibilidad y normativa | Casi todo | Copias cifradas y firmadas con GPG; un apartado de RGPD en la memoria |

Cada mes hay un hito en OpenProject, «Revisión de contenidos nuevos», en el que el grupo repasa lo visto y decide qué se engancha y qué no.

## Notas

- Documentar bien cada paso, con texto y capturas, como un tutorial que pueda seguirse copiando y pegando.
- Grabar vídeos de la demostración, por si el día de la defensa se cae algo.
- Pasos de los certificados: crear una CA propia con openssl, generar clave y petición por cada web (con el nombre en subjectAltName), firmar los certificados, configurar nginx en el puerto 443, instalar la CA en el portátil del técnico y probar desde allí.
- En la presentación diremos que conocemos las limitaciones de nginx en Windows Server y que, si la empresa cliente crece, incorporará un servidor específico para la web o contratará un servicio externo.

## Historial de versiones

| Versión | Fecha | Cambios |
| --- | --- | --- |
| 0.4 | 17/10/2026 | Borrador inicial completo |
| 0.5 | 18/10/2026 | Windows Server 2025, PostgreSQL en las sedes y Oracle Free en nuestra empresa, sede B en equipo de un compañero o portátil del centro, recursos de las MV, plantillas con Sysprep, nuevos riesgos y contenidos por ver |
| 0.6 | 19/10/2026 | Documentación dividida en un README breve y la carpeta `docs/`. Oracle en las sedes (visto bueno de ASGBD) con PostgreSQL como plan B; Fast-Start Failover en lugar de Patroni; más RAM para el servidor A y para OpenProject; bloque de IPE II en la defensa y reparto de tiempos; la restauración de la copia queda por decidir; el servidor del centro, previsto en un par de semanas |
| 0.7 | 19/10/2026 | Proyecto aprobado por el grupo. Sede B, casi seguro, en el PC de un compañero. Despliegue de equipos solo documentado: FOG para el taller y para las sedes, y Windows Autopilot como alternativa moderna de Microsoft (WDS se retira). Manuales numerados por orden de uso |
