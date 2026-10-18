# Manual: conectarse a OpenProject por VPN

Para: compañeros del grupo y docentes · Versión 1.1 · 18 de octubre de 2026

Mientras el centro no ceda su servidor, OpenProject (OP) funciona en una máquina virtual del PC de Jordi. Para llegar a ella desde fuera de su casa se usa una VPN con WireGuard: un túnel privado y cifrado entre tu equipo y esa máquina. Este manual explica cómo instalarla y conectarte, en Windows, en Linux y en el móvil.

> [!IMPORTANT]
> **Solo pasa por el túnel lo que va a OP.** Tu navegación normal, tu correo y lo demás siguen yendo por tu conexión de siempre: nadie ve tu tráfico ni se ralentiza tu internet.

## Por qué usamos vpn para esto  
Por cuatro motivos, de más práctico a más de fondo:

Sin abrir algo, nadie de fuera llega a la MV. OP vive en casa, detrás del router. Para que los compañeros entren hay dos caminos: abrir el puerto de OP (8080) directamente a internet, o abrir una VPN. Y la primera es mala idea por lo siguiente.  
Seguridad. Una web expuesta a internet recibe visitas de bots a las pocas horas, probando contraseñas y fallos conocidos. Además, OP va por HTTP sin certificado, así que las contraseñas viajarían en claro. Con WireGuard, en cambio:  
- Solo se abre un puerto, y WireGuard ni siquiera responde a quien no tenga una clave válida: para un escáner, ese puerto parece cerrado.  
- Todo el tráfico va cifrado dentro del túnel, aunque OP use HTTP.  
- Cada dispositivo tiene su propia clave, que puedes anular sin afectar a nadie más.  
- Y suma una segunda capa: aunque alguien robe un perfil, sigue necesitando un usuario de OP para entrar.  

Coherencia con el proyecto. En el README se indica que OP no se expondría a internet y que todo se administraría por VPN: las sedes, el técnico de la demo, los comerciales con el móvil. Montarlo para OP es practicar a pequeña escala lo mismo que haremos en el proyecto, y es justo lo que un profesor de SAD querrá ver.
La mudanza. Con la VPN, la dirección de OP (10.8.0.1:8080) y los perfiles pueden seguir igual cuando se pase al servidor del centro.

## Lo que necesitas

- **Tu fichero de perfil** (`algo.conf`), que te entrega Jordi en persona, en un USB. Cada dispositivo tiene el suyo: el de casa no sirve para el PC de clase, ni al revés.
- **Tu usuario y contraseña de OP**, que también te da Jordi.
- **Permisos de administrador** en el ordenador, para instalar WireGuard y activar el túnel.

## En el ordenador (Windows)

1. Descarga WireGuard desde su página oficial, `https://www.wireguard.com/install/`, e instálalo. Pedirá permisos de administrador.
2. Copia tu fichero de perfil a una carpeta de tu equipo que **no se sincronice con la nube** (ni OneDrive ni similares).
3. Abre WireGuard y pulsa **Importar túnel desde archivo**. Elige tu fichero `.conf`.
4. Pulsa **Activar**. El estado pasa a «Activo».
5. Abre el navegador y entra en:

   ```
   http://10.8.0.1:8080
   ```

6. Inicia sesión con tu usuario de OP.

Cuando termines, puedes pulsar **Desactivar**. No pasa nada por dejarlo activo, pero así sabes en todo momento cuándo estás conectado.

Una vez importado el perfil, puedes **borrar el fichero `.conf`** de la carpeta donde lo copiaste: WireGuard ya guarda su propia copia.

## En el móvil (Android o iPhone)

1. Instala la app **WireGuard** desde la tienda oficial.
2. Pulsa **+** y elige **Importar desde archivo** (o **Escanear código QR**, si Jordi te lo enseña en pantalla).
3. Activa el túnel con el interruptor.
4. En el navegador, escribe la dirección **completa, con `http://` delante**:

   ```
   http://10.8.0.1:8080/
   ```

5. Guárdala en la pantalla de inicio para no tener que teclearla más.

## En el ordenador (Linux)

> [!NOTE]
> Esta sección aún no se ha probado en nuestro entorno. Si la sigues, cuenta cómo te ha ido para confirmarla o corregirla.

Hay dos formas: por terminal, que funciona en cualquier distribución, o integrada en el gestor de red del escritorio. Los comandos son para distribuciones basadas en Debian o Ubuntu; en otras, cambia `apt` por el gestor de paquetes que toque.

### Opción A: por terminal (wg-quick)

1. Instala WireGuard:

   ```bash
   sudo apt install -y wireguard
   ```

2. Copia tu perfil a la carpeta de WireGuard con un nombre corto, sin espacios y de 15 caracteres como mucho, porque el nombre del fichero será el de la interfaz de red. Por ejemplo, `op.conf`. El comando `install` lo deja además con permisos solo para root:

   ```bash
   sudo install -m 600 tu-perfil.conf /etc/wireguard/op.conf
   ```

3. Activa el túnel y compruébalo:

   ```bash
   sudo wg-quick up op
   sudo wg show
   ```

   En `wg show` debe aparecer un **latest handshake** reciente.

4. Abre el navegador en `http://10.8.0.1:8080`.
5. Para desactivarlo:

   ```bash
   sudo wg-quick down op
   ```

Si quieres que se active solo al encender el equipo:

```bash
sudo systemctl enable --now wg-quick@op
```

Una vez copiado a `/etc/wireguard`, puedes borrar el fichero original de tu carpeta personal.

### Opción B: desde el gestor de red del escritorio (NetworkManager)

En escritorios como GNOME o KDE, el túnel puede aparecer junto al wifi y activarse con un clic:

```bash
sudo apt install -y wireguard-tools
nmcli connection import type wireguard file tu-perfil.conf
```

La conexión se crea con el nombre del fichero (sin `.conf`). A partir de ahí, se activa y desactiva desde el menú de red del escritorio, o por terminal:

```bash
nmcli connection up tu-perfil
nmcli connection down tu-perfil
```

Por defecto, NetworkManager activa la conexión sola al arrancar. Si prefieres activarla a mano, desactiva **Conectar automáticamente** en sus ajustes.

## Problemas frecuentes

| Qué pasa | Por qué | Qué hacer |
| --- | --- | --- |
| La página no carga y WireGuard no muestra «Último handshake» | La máquina de Jordi está apagada, o su IP de casa acaba de cambiar | Desactiva y activa el túnel. Si sigue igual, avisa a Jordi |
| El navegador del móvil busca la dirección en Google en lugar de abrirla | No ha entendido que es una dirección | Escríbela completa con `http://` y pulsa Intro en el teclado, no la sugerencia. Revisa que el corrector no haya metido espacios |
| El navegador avisa de que la página no es segura (HTTPS) | Dentro de la VPN aún no hay certificado | Continúa por HTTP. El tráfico ya va cifrado por el túnel |
| Funcionaba y deja de funcionar en otro equipo | Se está usando el mismo perfil en dos equipos a la vez | Cada equipo necesita su propio perfil. Pídeselo a Jordi |
| No conecta desde la red del centro | La red puede bloquear el tráfico de la VPN | Prueba con los datos del móvil. Si con datos funciona, es la red del centro |
| Has olvidado la contraseña de OP | — | La VPN sigue funcionando. Pide a Jordi que te la restablezca |

## Seguridad

- **Tu perfil es como una llave de casa:** no lo compartas, no lo subas a GitHub ni lo mandes por el grupo de clase.
- **Si pierdes el dispositivo o crees que alguien tiene tu perfil**, avisa a Jordi. Lo anula en un momento sin afectar a nadie más y te da uno nuevo.
- **La VPN y OP son dos llaves distintas:** el perfil deja llegar a la puerta de OP, pero para entrar hace falta además tu usuario y contraseña.

## Cuando OP se mude al servidor del centro

Si todo sale según lo previsto, **no tendrás que hacer nada**: la dirección (`http://10.8.0.1:8080`) y tu perfil seguirán valiendo. Si hiciera falta cambiar algo, Jordi te pasará un perfil nuevo y solo tendrás que importarlo igual que el primero.

## Disponibilidad

Mientras OP viva en el PC de Jordi, solo estará disponible cuando su PC y la máquina virtual estén encendidos. Si necesitas entrar en un horario concreto, avísale.
