# Manual (provisional): mudanza de OpenProject al servidor del centro

[← Índice de manuales](README.md)

Versión 0.1 · 9 de noviembre de 2026 · **Borrador: pendiente de que el centro entregue el servidor**

Este manual recoge cómo pasar OpenProject (OP) de la máquina virtual del PC de Jordi al servidor Proxmox del centro, con el menor impacto posible para quienes ya lo usan. Parte de lo que ya está montado (ver los manuales [10](10-montaje-vpn-openproject.md) y [20](20-conexion-vpn-openproject.md)).

> [!WARNING]
> Nada de lo marcado con `[PENDIENTE]` está confirmado, y los comandos se validarán el día de la mudanza. No sigas estos pasos hasta que el manual pase a la versión 1.0.

## Qué se mantiene y qué cambia

| Se mantiene (los usuarios no tocan nada) | Cambia (solo lo hace quien administra) |
| --- | --- |
| La red del túnel `10.8.0.0/24` y la dirección de OP, `http://10.8.0.1:8080` | La máquina donde vive OP y WireGuard |
| Las claves de WireGuard y los perfiles de todos | A dónde apunta el nombre de DuckDNS: a la IP del centro |
| La versión de OP (17.9.2) | El reenvío del puerto UDP 51820, que pasa a hacerlo el centro |

Esto funciona porque los perfiles apuntan a un **nombre** y no a una IP, y porque se trasladan las claves de WireGuard.

## Qué necesitamos del centro

- [PENDIENTE] La entrega del servidor Proxmox y un usuario propio con recursos suficientes para una MV de OP (como mínimo, 2 núcleos, 6 GB de RAM y 40 GB de disco).
- [PENDIENTE] Una IP fija para la MV y su salida a internet.
- [PENDIENTE] Que se permita la **entrada por UDP 51820** hacia esa MV. Es el único puerto que hay que abrir. Si el centro no puede, plan B: dejar el concentrador de la VPN en casa de Jordi y mudar solo OP.
- [PENDIENTE] Si hay copias o instantáneas de las MV en el centro.

## Pasos

1. **Preparar la MV nueva.** Ubuntu Server con Docker Engine y Docker Compose, igual que la actual, y la misma versión de OP (17.9.2). Activar el cortafuegos permitiendo solo UDP 51820 desde fuera y SSH desde la red de casa y el túnel (ver el apartado «Pendiente» del manual 10).
2. **Copia de la MV actual.** Antes de tocar nada, exportarla como OVA o hacer una instantánea. Es el plan de vuelta atrás.
3. **Copia de los datos de OP.** En la MV actual, con OP en marcha, volcar la base de datos (`pg_dump`) y copiar el volumen de archivos adjuntos. `[PENDIENTE]` Comprobar los nombres exactos del servicio y del volumen en el `docker-compose.yml` y el `.env` de la receta oficial.
4. **Parar OP en la MV actual** para que nadie escriba entre la copia y la mudanza. Avisar al grupo y a los docentes.
5. **Llevar los datos a la MV nueva y restaurarlos.** Levantar OP con la misma receta y comprobar que entra, que están los proyectos y las tareas, y que los adjuntos se ven.
6. **Trasladar WireGuard.** Copiar `/etc/wireguard` a la MV nueva (por un canal seguro: no por correo ni por GitHub) y arrancar `wg-quick@wg0`. La interfaz conserva `10.8.0.1`.
7. **Cambiar el nombre de DuckDNS.** Detener el cron de DuckDNS en la MV vieja y ponerlo en la nueva, o apuntar el nombre a la IP del centro desde la página de DuckDNS.
8. **Comprobar desde fuera.** Con tu perfil de siempre, la VPN debe mostrar un «último handshake» reciente (`sudo wg show` en la MV) y `http://10.8.0.1:8080` debe abrir OP. Que lo compruebe también un docente.
9. **Dejar la MV vieja apagada, pero sin borrarla**, unos días. Si algo falla, se vuelve atrás apuntando DuckDNS a la IP de casa y arrancándola.
10. **Programar la copia de seguridad nocturna** de la base de datos y de los adjuntos en la MV nueva, y traer una copia al PC de Jordi por la VPN.

## Qué pasa con las sedes

Las sedes entran al concentrador por el mismo nombre de DuckDNS, así que la mudanza no les afecta, siempre que los perfiles apunten al nombre. El túnel directo entre sedes (A-B) es independiente y no se toca.

## Historial

| Versión | Fecha | Cambios |
| --- | --- | --- |
| 0.1 | 09/11/2026 | Borrador inicial, a la espera de la entrega del servidor |
