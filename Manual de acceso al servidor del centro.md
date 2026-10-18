# Manual (provisional): acceso al servidor del centro desde casa

Versión 0.1 · 18 de octubre de 2026 · **Borrador: pendiente de las respuestas del centro**

Este manual se completará cuando el centro ceda el servidor y diga cómo se accede a él. Por ahora recoge lo que ya sabemos, lo que hay que preguntar y la estructura que tendrá.

> [!WARNING]
> Nada de lo marcado con `[PENDIENTE]` está confirmado. No sigas estos pasos hasta que el manual pase a la versión 1.0.

## Qué vamos a necesitar

Para trabajar en el servidor desde casa no hace falta ninguna máquina virtual en tu equipo: todo se hace desde tu Windows.

| Para qué | Herramienta | Ya disponible en Windows |
| --- | --- | --- |
| Llegar a la red del centro | La VPN que nos indique el centro `[PENDIENTE]` | No: habrá que instalarla |
| Administrar las máquinas virtuales (crear, arrancar, ver su pantalla) | Consola web de Proxmox, en el navegador | Sí |
| Trabajar en el servidor Linux de nuestra empresa | SSH | Sí (`ssh usuario@ip` en PowerShell) |
| Usar OP, las webs y Uptime Kuma | Navegador | Sí |

## Preguntas para el centro

- [ ] ¿Cómo se accede desde fuera a la red del centro? ¿Hay una VPN propia? ¿Cuál y cómo se configura?
- [ ] ¿Tendremos usuario propio en Proxmox? ¿Con qué permisos y qué cuotas de RAM, CPU y disco?
- [ ] ¿Podemos crear redes virtuales propias (para las cuatro redes de nuestra empresa)?
- [ ] ¿Se puede abrir el puerto UDP 51820 desde fuera hacia nuestra máquina? Lo necesitamos para mantener la VPN de OP sin cambiar los perfiles de nadie.
- [ ] ¿El servidor tiene IP pública fija? ¿Cuál?
- [ ] ¿Hay copias de seguridad o instantáneas de las máquinas virtuales?
- [ ] ¿Hay horarios de apagado o mantenimiento? En el centro hay cortes de luz frecuentes: ¿el servidor tiene SAI?

## Pasos previstos

### 1. Conectarse a la red del centro `[PENDIENTE]`

Instalar y configurar la VPN que indique el centro. Se completará con su procedimiento.

### 2. Entrar en la consola web de Proxmox `[PENDIENTE]`

En el navegador:

```
https://IP-DEL-SERVIDOR:8006
```

- La primera vez, el navegador avisará de que el certificado no es de confianza (Proxmox usa uno propio). Es normal en una red interna: se acepta la excepción.
- Usuario y dominio de inicio de sesión (`Realm`): los que dé el centro.

### 3. Entrar por SSH en nuestro servidor Linux `[PENDIENTE]`

En PowerShell:

```powershell
ssh usuario@IP-DE-NUESTRA-MV
```

Desde el primer día, acceso solo con clave (ed25519) y sin contraseña, como en la práctica de SSH.

### 4. Acceso a OP una vez migrado

Si el centro permite el puerto UDP 51820 (ver preguntas), la idea es que **todo siga igual para los usuarios**:

1. Copiar `/etc/wireguard` de la MV de OP de casa al servidor del centro.
2. Mantener la dirección `10.8.0.1` y el puerto 51820.
3. Detener el cron de DuckDNS en la MV de casa y apuntar el nombre a la IP del centro.
4. Migrar los datos de OP (base de datos y adjuntos) a una instalación con la misma versión, 17.9.2.
5. **Activar el cortafuegos (ufw) antes de dar acceso a nadie**, con «denegar por defecto» y abriendo solo UDP 51820 desde fuera y SSH desde el túnel. Ojo: Docker añade sus propias reglas y se salta ufw, así que el puerto de OP hay que publicarlo solo en la dirección del túnel (`PORT=10.8.0.1:8080` en el `.env`) y comprobarlo con un escáner de puertos desde fuera.

Resultado esperado: los usuarios siguen entrando en `http://10.8.0.1:8080` con el mismo perfil, sin tocar nada. Si el centro no permite ese puerto, se repartirán perfiles nuevos.

## Historial

| Versión | Fecha | Cambios |
| --- | --- | --- |
| 0.1 | 18/10/2026 | Borrador inicial, a la espera de las respuestas del centro |
