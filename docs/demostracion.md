# La demostración y el acceso remoto

[← Volver al README](../README.md)

## Reparto del tiempo

La defensa dura como máximo 40 minutos. Este es el reparto propuesto:

| Bloque | Minutos |
| --- | --- |
| IPE II: cómo nació nuestra empresa (preliminar, ver [IPE II](ipe-ii.md)) | 8-10 |
| Diseño de la infraestructura: diagrama y decisiones | 6-8 |
| Demo: el técnico entra en las dos empresas | 6-8 |
| Demo: cae la sede A y la B toma el relevo | 5-6 |
| Limitaciones, mejoras y cierre | 3-4 |
| Margen para imprevistos y preguntas | 4-5 |

## Guion de la demostración

Guion propuesto, aún por pulir:

1. **Estado de partida.** Panel de Uptime Kuma con toda la infraestructura en verde (servidores, VPN, webs y base de datos) y un vistazo a las dos webs desde el portátil del técnico.
2. **El técnico entra en nuestra empresa.** Conecta por VPN, abre una sesión SSH con clave, accede por SFTP a la carpeta de herramientas portables y consulta en la intranet el historial de reparaciones de un cliente. No ve nada más.
3. **El técnico entra en la empresa cliente.** Desde el mismo portátil, escritorio remoto al servidor y al equipo de oficina de la sede, con un usuario de un departamento y sus propios permisos.
4. **Cae la sede A.** Motivo por decidir: el gato que pulsa el botón de apagado o similar. Se simula con un apagado brusco de la máquina virtual. Uptime Kuma marca en rojo la sede A. El técnico promueve la copia de la sede B con un comando (con el chiste del administrador al que despiertan a las tres de la mañana) y la web sigue funcionando desde ahí. Si llegamos a montar Fast-Start Failover, la promoción la hace sola el observador.
5. **El técnico comprueba que B sigue sirviendo.** Y llama a la sede para que echen al gato.
6. **Si da tiempo.** Una compra de prueba en la web del cliente, para ver cómo cambia la base de datos.

> [!NOTE]
> **Pendiente de decidir:** si se demuestra la restauración de una copia de seguridad y cómo, porque el tiempo es limitado. La idea inicial era restaurar una carpeta pequeña desde un USB conectado a la MV del servidor de la sede A.

## Cómo se controla todo a distancia

El portátil entra por la VPN del proyecto. A las máquinas Windows de cada sede entra por escritorio remoto (RDP); desde un portátil con Linux sirve un cliente como Remmina. Las máquinas virtuales Windows deben ser Pro o Enterprise, y los equipos anfitriones también para entrar a ellos: el de la sede A es Pro y el de la B hay que comprobarlo (la edición Home no acepta escritorio remoto entrante; la Education, sí). Al servidor Linux se entra por SSH y por la consola web de Proxmox; a las webs, a Uptime Kuma y a OpenProject, con el navegador. Para apagar la máquina virtual de la sede A, escritorio remoto al PC de Jordi y la línea de comandos de VirtualBox. No usaremos TeamViewer ni otros servicios parecidos: sacan la conexión por su nube y contradicen el diseño sobre VPN.

**El USB en la máquina virtual** (si se hace la restauración). En VirtualBox, los puertos USB 2.0 y 3.0 piden el Extension Pack, así que hay que probarlo con tiempo.

**Qué simulamos de nuestra empresa.** No montamos la tienda, el taller ni la red de invitados: se diseñan y se documentan (esquema y tabla de reglas del firewall). Solo simulamos el acceso de un técnico que está fuera: con el portátil prestado por el centro, consulta por VPN los datos de su cliente en nuestro servidor y, desde ese mismo portátil, entra en el servidor de la empresa cliente.

**El portátil.** Es del centro, con Linux o con Windows 11, y habrá que preguntar a quien gestione esos portátiles. Va conectado a un proyector o pantalla grande: no se enseñan contraseñas ni claves, y se prueba la resolución antes del día de la presentación.

## Comprobación del día antes y del mismo día

Las IP de casa cambian, así que cada uno:

1. Anota su IP pública.
2. La compara con la que hay en la configuración del túnel.
3. La corrige si no coincide y reinicia el túnel.
4. Comprueba que hay un «último handshake» reciente (en WireGuard, con `wg show`).
5. Hace ping a la otra sede por la dirección del túnel.
6. Repite todo antes de entrar a la defensa.

Ensayo general unos días antes, y el móvil como punto de acceso si la red del centro bloquea la VPN. Vídeos grabados de cada demostración clave, por si algo falla el día de la defensa.
