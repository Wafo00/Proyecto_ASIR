# Manual: montar una VPN WireGuard para acceder a OpenProject

[← Índice de manuales](README.md)

Versión 1.0 · 18 de octubre de 2026 · Probado y funcionando

Este manual documenta cómo se montó el acceso remoto a OpenProject (OP) mientras está alojado en una máquina virtual del PC de Jordi. Todos los pasos están probados, salvo los marcados como **pendientes de probar**.

## Diseño

| Elemento | Valor |
| --- | --- |
| Máquina virtual de OP | Ubuntu Server 26.04 en VirtualBox, adaptador puente, IP fija `192.168.1.20` (fuera del rango DHCP del router, que va de `.33` a `.199`) |
| Servidor VPN | WireGuard en la propia MV de OP, interfaz `wg0` |
| Puerto | UDP `51820`, reenviado en el router hacia `192.168.1.20` |
| Red del túnel | `10.8.0.0/24`. La MV es `10.8.0.1`; cada dispositivo cliente tiene su propia IP |
| Dirección de OP para todos | `http://10.8.0.1:8080` |
| Nombre público | DNS dinámico con DuckDNS (`TUSUBDOMINIO.duckdns.org`), actualizado desde la MV cada 5 minutos |
| Tipo de túnel | Dividido: los clientes solo mandan por el túnel el tráfico hacia `10.8.0.1`. Su internet no pasa por la VPN, y la MV no necesita enrutar ni hacer NAT |

**Por qué así:**

- **Solo se abre un puerto en casa, el de WireGuard.** OP (8080) y SSH (22) no se exponen a internet: solo se llega a ellos por dentro del túnel.
- **Los perfiles apuntan a un nombre, no a una IP.** Si la IP pública de casa cambia, no hay que tocar ningún perfil.
- **Pensado para la mudanza al servidor del centro:** si se trasladan las claves de WireGuard, se conserva la dirección `10.8.0.1` y el nombre de DuckDNS pasa a apuntar al centro, los dispositivos de los usuarios no necesitan ningún cambio.

### Reparto de direcciones del túnel

| Bloque | Para quién |
| --- | --- |
| `10.8.0.1` | La MV de OP (servidor) |
| `10.8.0.2` a `10.8.0.19` | Alumnos |
| `10.8.0.20` a `10.8.0.39` | Docentes |
| `10.8.0.40` en adelante | Libre (portátil de la demo, imprevistos) |

## 1. Comprobar que la conexión no está detrás de CGNAT

Con CGNAT, el operador comparte la IP pública entre varios clientes y no se pueden abrir puertos, así que este montaje no funcionaría.

1. Entrar en el router (`http://192.168.1.1`) y anotar la **IP WAN**.
2. Abrir `https://ifconfig.me` y anotar la IP que muestra.
3. Si coinciden, no hay CGNAT. Si son distintas, o la del router empieza por `100.64` a `100.127`, hay CGNAT.

Resultado: **sin CGNAT**.

## 2. Instalar WireGuard y crear las claves del servidor

En la MV, por SSH:

```bash
sudo apt install -y wireguard
sudo bash -c 'umask 077; cd /etc/wireguard && wg genkey | tee server.key | wg pubkey > server.pub'
sudo ls -l /etc/wireguard
```

Deben aparecer `server.key` y `server.pub` con permisos `-rw-------` y propietario `root`. `umask 077` hace que los ficheros nazcan ya legibles solo por root.

> [!CAUTION]
> La clave privada (`server.key`) no sale nunca de la MV: ni a GitHub, ni a OP, ni a ningún chat. La pública (`server.pub`) va dentro de los perfiles de los clientes.

## 3. Configurar y arrancar el servidor

Crear `/etc/wireguard/wg0.conf` con la clave privada dentro, sin que llegue a mostrarse en pantalla:

```bash
sudo bash -c 'umask 077; printf "[Interface]\nAddress = 10.8.0.1/24\nListenPort = 51820\nPrivateKey = %s\n" "$(cat /etc/wireguard/server.key)" > /etc/wireguard/wg0.conf'
sudo grep -v PrivateKey /etc/wireguard/wg0.conf
```

Arrancarlo y hacer que se inicie con la MV:

```bash
sudo systemctl enable --now wg-quick@wg0
sudo wg show
ip a show wg0
```

`ip a show wg0` debe mostrar `inet 10.8.0.1/24`.

Cortafuegos: `sudo ufw status` devolvió `inactive`, así que no interfiere. Activarlo queda como mejora pendiente (ver «Pendiente»).

## 4. Reenviar el puerto en el router

Router de Movistar (modelo GPT-2741GNAC): **NAT > Port Forwarding > Add new rule**.

| Campo | Valor |
| --- | --- |
| Active | Marcado |
| Service Name | User Define |
| External Start Port / External End Port | `51820` / `51820` |
| Server IP Address | `192.168.1.20` |
| Protocol | **UDP** (no «ALL») |
| Open Start Port / Open End Port | `51820` / `51820` |

No se abre ningún otro puerto. El puerto TCP 7547 lo reserva Movistar para gestionar el router: no tocarlo.

## 5. DNS dinámico con DuckDNS

1. En `https://www.duckdns.org`, iniciar sesión con la cuenta de GitHub y crear un subdominio genérico (sin nombres de empresa). Anotar el **token**, que es secreto.
2. En la MV, crear el script de actualización (sustituir `TUSUBDOMINIO` y `TUTOKEN`; en `domains` va el subdominio sin `.duckdns.org`):

   ```bash
   mkdir -p ~/duckdns
   echo 'curl -s "https://www.duckdns.org/update?domains=TUSUBDOMINIO&token=TUTOKEN&ip=" -o ~/duckdns/duck.log' > ~/duckdns/duck.sh
   chmod 700 ~/duckdns/duck.sh
   ```

3. Probarlo: debe responder `OK` (si responde `KO`, el subdominio o el token están mal).

   ```bash
   ~/duckdns/duck.sh && cat ~/duckdns/duck.log; echo
   ```

4. Programarlo cada 5 minutos con cron:

   ```bash
   (crontab -l 2>/dev/null; echo '*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1') | crontab -
   crontab -l
   ```

El intervalo es el tiempo máximo que el nombre puede apuntar a la IP vieja tras un cambio. Cada ejecución es una petición mínima, y DuckDNS recomienda 5 minutos; 15 también sería razonable.

## 6. Script para dar de alta perfiles

`~/wg-alta.sh` genera las claves de un cliente, lo da de alta en el servidor (en caliente y en `wg0.conf`, para que sobreviva a reinicios) y crea su perfil en `/etc/wireguard/clientes/`, accesible solo para root.

```bash
sudo apt install -y qrencode
nano ~/wg-alta.sh
```

Contenido (sustituir `TUSUBDOMINIO`):

```bash
#!/bin/bash
# Uso: ./wg-alta.sh NOMBRE IP     (ejemplo: ./wg-alta.sh movil-jordi 10.8.0.2)
set -euo pipefail
NOMBRE="$1"
IP="$2"
ENDPOINT="TUSUBDOMINIO.duckdns.org:51820"
DIR=/etc/wireguard/clientes

sudo mkdir -p "$DIR" && sudo chmod 700 "$DIR"
if [ -e "$DIR/$NOMBRE.conf" ]; then echo "Ya existe un perfil llamado $NOMBRE"; exit 1; fi

PRIV=$(wg genkey)
PUB=$(echo "$PRIV" | wg pubkey)
SRVPUB=$(sudo cat /etc/wireguard/server.pub)

# Alta en el servidor: en caliente y en el fichero, para que sobreviva a reinicios
sudo wg set wg0 peer "$PUB" allowed-ips "$IP/32"
printf '\n[Peer]\n# %s\nPublicKey = %s\nAllowedIPs = %s/32\n' "$NOMBRE" "$PUB" "$IP" | sudo tee -a /etc/wireguard/wg0.conf > /dev/null

# Perfil del cliente: solo el tráfico hacia la MV va por el túnel
printf '[Interface]\nPrivateKey = %s\nAddress = %s/32\n\n[Peer]\nPublicKey = %s\nEndpoint = %s\nAllowedIPs = 10.8.0.1/32\nPersistentKeepalive = 25\n' \
  "$PRIV" "$IP" "$SRVPUB" "$ENDPOINT" | sudo tee "$DIR/$NOMBRE.conf" > /dev/null
sudo chmod 600 "$DIR/$NOMBRE.conf"

echo "Perfil creado: $DIR/$NOMBRE.conf"
```

```bash
chmod 700 ~/wg-alta.sh
```

Claves del diseño del perfil:

- `AllowedIPs = 10.8.0.1/32`: túnel dividido; solo el tráfico hacia OP va por la VPN.
- `PersistentKeepalive = 25`: mantiene viva la conexión a través del NAT del router del cliente.
- **Un perfil por dispositivo:** cada perfil es una clave; si dos equipos usan el mismo a la vez, se desconectan mutuamente.

## 7. Crear los perfiles

```bash
cd ~
./wg-alta.sh movil-jordi      10.8.0.2
./wg-alta.sh clase-jordi      10.8.0.3
./wg-alta.sh companero1-casa  10.8.0.4
./wg-alta.sh companero1-clase 10.8.0.5
./wg-alta.sh companero2-casa  10.8.0.6
./wg-alta.sh companero2-clase 10.8.0.7
./wg-alta.sh docente-sri      10.8.0.20
./wg-alta.sh docente-iaw      10.8.0.21
./wg-alta.sh docente-sad      10.8.0.22
./wg-alta.sh docente-aso      10.8.0.23
./wg-alta.sh docente-asgbd    10.8.0.24
```

Comprobación: once ficheros y once *peers*.

```bash
sudo ls /etc/wireguard/clientes
sudo wg show wg0 | grep -c peer
```

## 8. Prueba desde fuera de casa

1. Mostrar el QR del perfil del móvil:

   ```bash
   sudo bash -c 'qrencode -t ansiutf8 < /etc/wireguard/clientes/movil-jordi.conf'
   ```

2. En el móvil: app WireGuard > **+** > **Escanear código QR**.
3. Desactivar el wifi (para entrar por datos móviles, como desde fuera) y activar el túnel.
4. En la MV, `sudo wg show`: el peer `10.8.0.2/32` debe mostrar un **latest handshake** reciente.

Resultado: **handshake correcto desde datos móviles**.

## 9. Sacar los perfiles a un USB seguro

En la MV:

```bash
sudo tar czf ~/perfiles-wg.tar.gz -C /etc/wireguard clientes
sudo chown $USER: ~/perfiles-wg.tar.gz && chmod 600 ~/perfiles-wg.tar.gz
```

En PowerShell de Windows (en una carpeta que **no** esté sincronizada con OneDrive):

```powershell
mkdir $HOME\perfiles-temp; cd $HOME\perfiles-temp
scp jordi@192.168.1.20:perfiles-wg.tar.gz .
tar -xzf perfiles-wg.tar.gz
```

Copiar la carpeta `clientes` al USB cifrado y borrar las copias temporales:

```powershell
cd $HOME; Remove-Item -Recurse -Force $HOME\perfiles-temp
```

```bash
rm ~/perfiles-wg.tar.gz
```

Los originales se quedan en `/etc/wireguard/clientes` de la MV, solo accesibles para root. Los perfiles se entregan **en persona**, nunca por GitHub ni por el grupo de clase.

## 10. OP en la dirección del túnel

Cambiar la dirección de OP en su `.env` y aplicar:

```bash
cd ~/openproject
sed -i 's|^OPENPROJECT_HOST__NAME=.*|OPENPROJECT_HOST__NAME=10.8.0.1:8080|' .env
sed -i 's|^COLLABORATIVE_SERVER_URL=.*|COLLABORATIVE_SERVER_URL=ws://10.8.0.1:8080/hocuspocus|' .env
grep -E "HOST__NAME|COLLABORATIVE_SERVER_URL" .env
docker compose up -d
docker compose ps web
```

Esperar a que `web` esté `(healthy)`.

En el PC de casa, que está en la misma red que la MV y no usa la VPN, añadir una ruta persistente hacia la red del túnel (PowerShell como administrador):

```powershell
route -p add 10.8.0.0 mask 255.255.255.0 192.168.1.20
ping 10.8.0.1
```

Funciona porque Linux responde por cualquier interfaz a las direcciones que tiene asignadas, así que la MV contesta a `10.8.0.1` aunque la petición le llegue por la red de casa.

Resultado: **OP carga en `http://10.8.0.1:8080` desde el PC de casa y desde el móvil por datos.**

## Dar de baja un perfil (pendiente de probar)

Si se pierde un dispositivo o un perfil deja de usarse:

1. Ver su clave pública en `wg0.conf` (cada bloque `[Peer]` lleva el nombre en un comentario):

   ```bash
   sudo grep -A2 "# NOMBRE" /etc/wireguard/wg0.conf
   ```

2. Quitarlo en caliente:

   ```bash
   sudo wg set wg0 peer CLAVE_PUBLICA remove
   ```

3. Borrar su bloque `[Peer]` de `/etc/wireguard/wg0.conf` (con `sudo nano`) y su fichero de `/etc/wireguard/clientes/`.

## Pendiente

- **Activar el cortafuegos (ufw)** en la MV, permitiendo solo UDP 51820 desde fuera y SSH desde la red de casa y el túnel. Ojo: Docker añade sus propias reglas y se salta ufw, así que el puerto 8080 de OP necesita un tratamiento aparte (por ejemplo, publicarlo solo en `10.8.0.1`).
- **Copiar tu clave pública SSH a la MV y desactivar el acceso por contraseña.**
- **Mudanza al servidor del centro:** copiar `/etc/wireguard`, conservar `10.8.0.1`, detener el cron de DuckDNS en esta MV y apuntar el nombre a la IP del centro. Depende de que el centro permita la entrada por UDP 51820.

## Problemas encontrados

- **Firefox para Android busca la dirección en lugar de abrirla:** hay que escribirla completa, `http://10.8.0.1:8080/`, y pulsar Intro en el teclado.
