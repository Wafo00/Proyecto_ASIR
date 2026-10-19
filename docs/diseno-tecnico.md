# Diseño técnico

[← Volver al README](../README.md)

## Contenido

1. [Nuestra empresa](#nuestra-empresa)
2. [La empresa cliente](#la-empresa-cliente)
3. [Infraestructura y ubicación](#infraestructura-y-ubicación)
4. [Servicios y para qué sirven](#servicios-y-para-qué-sirven)
5. [Sitio web y base de datos](#sitio-web-y-base-de-datos)
6. [Cómo se autentican los usuarios en el sitio web](#cómo-se-autentican-los-usuarios-en-el-sitio-web)
7. [Herramientas de gestión y documentación](#herramientas-de-gestión-y-documentación)
8. [Alcance por prioridades](#alcance-por-prioridades)

## Nuestra empresa

Somos una pequeña empresa de servicios informáticos con taller (reparación y montaje) que acaba de crecer para dar servicio a su primera empresa cliente. Además de reparar equipos, ahora diseñamos, desplegamos y mantenemos la infraestructura de otras empresas. El nombre está por decidir.

Para nosotros mismos contamos con:

- Un **pequeño servidor Linux**, alojado en el servidor del instituto, con la base de datos de clientes, trabajadores, proveedores, productos y reparaciones, y los servicios básicos: HTTPS, DNS, DHCP y copias de seguridad, además de otros que iremos aprendiendo.
- **5 terminales**: 4 estaciones de trabajo (dos en la tienda y dos en el taller) y un portátil de técnico, prestado por el centro, para simular el acceso desde fuera. En la tienda, un equipo hace de mostrador y otro de escritorio de presupuestos, donde está toda la información. Sistemas mezclados; los técnicos usarán Linux.
- **Un firewall virtual** (OPNsense o pfSense, por decidir) que separa cuatro redes: tienda, taller, equipos en reparación e invitados. Cada una llega al servidor solo a lo que necesita y entre ellas no hay acceso; los invitados solo salen a internet. El taller y los equipos en reparación tienen el mayor aislamiento posible, para que un virus no pase al portátil de un cliente.
- Pocos departamentos: especialistas en software y programación, y técnicos de reparación y montaje. Las gestiones (facturas, pagos, impuestos...) las lleva una gestoría externa.
- Una **web sencilla** de tarifas, servicios y contacto, con un pequeño blog de cuatro entradas (dos noticias ficticias y dos consejos prácticos). No recibe pedidos ni lleva base de datos. Se aloja en nginx con su propio certificado.

Seguridad y continuidad:

- **Copias de seguridad:** dos unidades físicas cifradas que se alternan. Una queda conectada al servidor y la otra sale del local, en manos del dueño o de un empleado de confianza.
- **SAI:** uno para el servidor y el equipo de la tienda, con tiempo suficiente para apagarlos de forma ordenada, y otro aparte para el taller. Un corte en mitad de una actualización de BIOS puede dejar una placa inservible.
- **Servidor seguro:** SSH solo con clave y solo por el túnel de la VPN, cortafuegos con «denegar por defecto» (ufw o nftables), actualizaciones automáticas y bloqueo de intentos fallidos (fail2ban). Ojo: Docker abre puertos por su cuenta y puede saltarse el cortafuegos. Se publican solo en la red interna y se comprueba con un escáner desde fuera.
- **Acceso del técnico a domicilio:** una VPN personal por técnico, SSH con clave y, en lugar de una consola abierta, una carpeta con herramientas portables de código abierto a la que accede por SFTP. Además consulta, desde la intranet web, los datos del cliente que necesita: el historial de reparaciones (equipo, avería, estado, garantía y contacto), con permiso de lectura y, como mucho, para añadir notas a la reparación. No ve facturación ni datos de otros clientes, y cada acceso queda registrado: hay datos personales, así que se aplica el mínimo necesario.
- **Instalación de equipos por red (FOG), solo documentado:** desplegar imágenes en los equipos del taller y reinstalarlos por red. Se explica en la memoria y en la defensa; no se monta.
- **Proxy de salida (Squid) para el taller**, opcional: filtra y registra la navegación por dominio, sin abrir el tráfico HTTPS.
- **Base de datos:** Oracle Free (la edición gratuita) en un contenedor Docker. Al ser un solo servidor sin réplica, la edición gratuita basta. Consume unos 2-3 GB de RAM, a tener en cuenta al pedir recursos en Proxmox.

## La empresa cliente

Nuestro primer cliente se dedica a la importación y distribución de artículos de regalo, y vende a tiendas y a clientes online. Tiene **dos sedes en grandes ciudades**, con entre 7 y 14 ordenadores cada una, repartidos en departamentos: comercial, import/export, diseño, contabilidad, RRHH, almacén y dirección.

Lo que necesita:

- **Identidad y permisos centralizados** en cada sede (Active Directory).
- **Conexión segura entre sedes** mediante VPN, con copias de seguridad.
- Una **zona de comerciales en el sitio web**, para los comerciales de calle: consultar el stock en tiempo real y ver el estado de pedidos e incidencias. Los comerciales no trabajan en las sedes: en el departamento comercial de cada sede hay encargados que coordinan a los comerciales de cada zona geográfica.
- Una **web para clientes**: catálogo, alta, modificar sus datos y hacer pedidos. Es la parte más compleja, porque exige datos siempre actualizados (stock, precios, productos, clientes). En la demostración la dejamos en lo mínimo: ver productos con su stock y crear un pedido.

Cómo es cada sede (la simulamos con dos máquinas virtuales):

- **Un servidor Windows Server 2025 que lo hace todo:** Active Directory, DHCP, DNS, carpetas compartidas, base de datos (Oracle) y web. Es lo habitual en una empresa pequeña, y nadie de la oficina lo toca: solo nosotros.
- **Un cliente Windows 11 Pro** unido al dominio, como perfil representativo, con los recursos mínimos. En la memoria explicaremos que los demás equipos (de 7 a 14 por sede) serían copias de ese perfil, con las mismas políticas (GPO).
- **Red:** el servidor tiene dos adaptadores, uno de red interna (la oficina) y otro en puente (salida a internet y a la VPN). Los trabajadores de la oficina solo tienen el interno y salen a través del servidor.
- **Seguridad de nivel 1:** cortafuegos de Windows con «denegar por defecto», perfil de red público en el adaptador externo, administración (RDP) solo por la VPN, cuentas de administrador separadas, directivas de contraseñas por GPO y bloqueo tras fallos.
- **Despliegue de equipos, solo documentado:** una imagen de Windows 11 con la suite ofimática y los programas de gestión, para instalar igual todos los equipos de la oficina. La herramienta clásica de Microsoft para esto, WDS, deja de incluirse después de Windows Server 2025, y su compañera MDT ya está retirada. Lo actual en el mundo Microsoft es **Windows Autopilot con Intune** (desde la nube, con licencias de Microsoft 365) o, en local, Configuration Manager, demasiado pesado para una empresa de este tamaño. Nuestra propuesta: desplegar la imagen con **FOG** desde nuestro servidor, que ya lo usamos para el taller y también despliega Windows, y explicar Autopilot como lo que haría el cliente si se pasara a Microsoft 365.

Seguridad y continuidad del cliente:

- **SAI en la sede A:** debe aguantar unas dos horas de corte y alimentar el servidor, la pantalla, el módem o router y el switch, para que sigan funcionando la VPN, la web y los comerciales de calle. Los equipos de oficina son prescindibles. El SAI será de gama media, con regulador de tensión (AVR) y onda senoidal pura, y apagado ordenado del servidor si el corte se alarga. La capacidad se calcula con el consumo medido (vatios por horas, con margen).
- **Copias de seguridad (solo en la documentación, no se montan):** una copia cruzada entre sedes por VPN durante la noche y una copia física automática en un disco conectado al servidor. Suponemos ambas sedes con al menos 1 Gbps contratado y entre 10 y 15 GB por copia completa.
- **Estrategia de copias:** en la física, una completa semanal y un incremental diario; en la cruzada, una completa cada noche. La base de datos se copia con la herramienta del propio motor (RMAN en Oracle), no copiando ficheros en caliente.
- **Buenas prácticas:** cifrar los discos, tener dos y alternarlos, sacar uno del local y desconectarlo del servidor para que un ransomware no lo cifre con el resto. Se sigue la regla 3-2-1 (tres copias, dos soportes y una fuera del sitio).

## Infraestructura y ubicación

Montamos todo en cuatro sitios, cada uno con un propósito:

- **Servidor del centro (Proxmox).** Es el único punto siempre encendido y con conexión estable. Aloja el único servidor Linux del proyecto, el de nuestra empresa, con Docker: su web, OpenProject, Uptime Kuma y la base de datos Oracle Free. Además lleva el firewall virtual y el concentrador de la VPN y, si se monta la promoción automática, el observador de la base de datos de la empresa cliente. Necesitamos que nuestro entorno quede aislado del de otros grupos. El docente está de acuerdo en cedérnoslo; previsto en un par de semanas.
- **Sede A: el equipo de Jordi (6 núcleos y 12 hilos, 32 GB).** Un solo PC con dos máquinas virtuales (servidor y cliente). Lleva la base de datos principal. También sirve de equipo de desarrollo y de plan B. Mientras no haya servidor en el centro, aloja además la MV de OpenProject.
- **Sede B: el PC de un compañero (casi seguro), con un portátil del centro (16 GB DDR4 y SSD SATA) como alternativa.** Estará en casa de ese compañero, con las mismas dos máquinas virtuales, la copia de la base de datos y la web en reserva. Sea cual sea el equipo, debe tener permisos de administrador, la virtualización activada en la BIOS y poder desactivar la seguridad basada en virtualización (VBS) de Windows.
- **Portátil de la demostración.** Un portátil prestado por el centro, conectado a un proyector o pantalla grande, que hace de técnico: entra por VPN y escritorio remoto a los servidores de cada sede y al de nuestra empresa. Si la sede B también va en un portátil del centro, serán dos portátiles distintos.

Las sedes y el portátil de la demo salen hacia el servidor del centro, así que no hace falta abrir puertos en casa de nadie, salvo para el túnel directo entre sedes, que sigue funcionando aunque el centro caiga. Es obligatorio por el diseño de la base de datos: la réplica viaja por él.

**Cómo se conectan las máquinas virtuales.** El servidor de cada sede hace de router de su oficina:

| Máquina virtual | Adaptador | Tipo | Para qué |
| --- | --- | --- | --- |
| Servidor | 1 | Red interna | La red de la oficina: DHCP, DNS, AD, archivos |
| Servidor | 2 | Puente | Salir a internet y a la VPN (centro y otra sede) |
| Cliente Windows | 1 | Red interna | Solo la oficina; sale a internet a través del servidor |

Hay cuatro cosas que configurar con cuidado:

1. **DHCP solo en la red interna**, para no competir con el router de casa y repartir direcciones a otros aparatos.
2. **DNS y Active Directory solo en la interfaz interna.** Hay que desactivar el registro DNS de la tarjeta externa.
3. **Reenvío y NAT** en el servidor (rol de Acceso remoto, enrutamiento y NAT). Sin ellos, los clientes llegan al servidor pero no salen.
4. **Ruta de vuelta.** El portátil del técnico debe saber que la red de la oficina está detrás del túnel, y los equipos de la oficina deben usar la IP interna del servidor como puerta de enlace.

**Recursos de las máquinas virtuales.** Discos dinámicos (solo ocupan lo que se usa) y EFI activado:

| Sede | Máquina virtual | Núcleos | RAM | Disco |
| --- | --- | --- | --- | --- |
| A (PC de Jordi, 32 GB) | Servidor `SEDEA-SRV01` | 4 | 8 GB | 80 GB |
| A | Cliente Windows 11 | 2 | 3 GB | 64 GB |
| A (provisional) | OpenProject (Ubuntu Server) | 2 | 6 GB | 40 GB |
| B (PC de un compañero; 16 GB como mínimo) | Servidor `SEDEB-SRV01` | 2-4, según el equipo | 6 GB | 80 GB |
| B | Cliente Windows 11 | 2 | 3 GB | 64 GB |

El servidor de la sede A lleva 8 GB porque Oracle es exigente con la memoria; el de la B, algo menos, porque su base de datos es la copia y su web está en reserva. En la sede A quedan unos 15 GB para el anfitrión y en la B unos 7. OpenProject necesita al menos 6 GB: con 4 GB tira de memoria de intercambio. Ninguna MV debe tener más núcleos de los que tiene físicamente su anfitrión. La red interna de cada sede debe llamarse igual en el servidor y en el cliente (por ejemplo, `sedeA-lan`).

**Antes de crear las MV en un anfitrión.** Windows 11 puede tener activo su propio hipervisor sin avisar (seguridad basada en virtualización, VBS, que activan la integridad de memoria o la protección de inicio de sesión de Windows Hello). Con él, VirtualBox funciona en modo lento (el icono de la tortuga verde) y algunas instalaciones se cuelgan. Hay que desactivarlo y comprobar en `msinfo32` que aparecen las líneas de «Requisitos de Hyper-V». El procedimiento paso a paso irá en el manual de montaje de las sedes.

**Plantillas.** Para no instalar dos veces lo mismo, el servidor y el cliente de la sede A, recién instalados, actualizados y en español (sin roles ni configuración propia de la sede), se generalizan con Sysprep (`sysprep /generalize /oobe /shutdown`) y se exportan como OVA. La sede B los importa con direcciones MAC nuevas y continúa desde ahí. Un controlador de dominio ya configurado nunca se clona.

**Licencias.** Se asumen licencias educativas para los Windows (Azure Dev Tools for Teaching o similar). Si no llegan, versión de evaluación de 180 días, ampliable con `slmgr /rearm`. Deben conseguirse antes de instalar AD: un controlador de dominio no puede convertirse de evaluación a versión con licencia.

**Direcciones.** Manejamos cuatro grupos de IP: la red real de cada casa, la IP del adaptador puente de cada servidor, la red interna de cada sede, y el túnel. Las dos redes internas deben ser distintas entre sí y distintas de la de casa, y la IP puente debe quedar fuera del rango DHCP del router de casa. Haremos una tabla de IPs desde el principio.

**IP dinámica.** En la empresa real, las sedes tendrían IP fija: la IP dinámica es una limitación de la simulación, porque las sedes estarán en casas. Los túneles hacia el centro no se ven afectados, pero el túnel directo A-B sí, y es obligatorio. Se usa DNS dinámico: el túnel apunta a un nombre y no a una IP. WireGuard resuelve el nombre al arrancar, así que si la IP cambia hay que reiniciar el túnel. Tampoco resuelve el CGNAT: hay que comprobar que ninguna sede está detrás de uno, comparando la IP que muestra el router con la que se ve en una web de «mi IP».

## Servicios y para qué sirven

La columna de módulos es una primera estimación y hay que confirmarla con los docentes.

| Servicio | Dónde | Para qué | Módulos (a confirmar) |
| --- | --- | --- | --- |
| Hipervisor Proxmox | Servidor del centro | Alojar nuestras máquinas virtuales, aisladas de otros grupos | SRI, ASO |
| Firewall virtual (OPNsense o pfSense) | Servidor del centro | Separar las cuatro redes de nuestra empresa y aislar los equipos en reparación | SAD |
| VPN (WireGuard) | Concentrador en el servidor del centro; sedes, portátil y móviles de los comerciales como clientes | Unir las sedes y permitir el acceso remoto seguro | SRI, SAD |
| DHCP y DNS | Nuestro servidor Linux y cada sede (Windows Server) | Direcciones y nombres de red | SRI |
| Active Directory | Cada sede | Usuarios, grupos por departamento y políticas (GPO) | ASO |
| Carpetas compartidas | Cada sede | Archivos por departamento, con permisos | ASO |
| Base de datos con copia (Oracle) | Windows Server de cada sede (principal en A, copia en B) | Datos de pedidos, stock y clientes con alta disponibilidad | ASGBD |
| Base de datos de nuestra empresa (Oracle Free) | Servidor Linux del centro (Docker) | Clientes, trabajadores, proveedores, productos y reparaciones | ASGBD |
| Observador de la base de datos, opcional | Servidor Linux del centro | Decidir la promoción automática si cae la sede A | ASGBD, SAD |
| Sitio web en Python con nginx | Cada sede (activa en A, en reserva en B) | Pedidos de comerciales y clientes con una sola base de datos | IAW |
| Web de nuestra empresa y blog | Servidor Linux del centro | Información y contacto, con su propio certificado | IAW |
| Nginx y certificados | Ambas empresas | HTTPS con certificados de openssl y una CA propia | IAW, SRI |
| OpenProject | Servidor Linux del centro (Docker); provisionalmente, en una MV del PC de Jordi | Planificar y repartir el trabajo del grupo | Contenedores |
| Uptime Kuma | Servidor Linux del centro (Docker) | Panel con el estado de toda la infraestructura y un script que avisa de que la copia del día se ha hecho | Contenedores, SAD |
| SFTP restringido a una carpeta | Servidor Linux de nuestra empresa | Que el técnico acceda a una carpeta con herramientas portables | ASO, SRI |
| FOG, solo documentado | Servidor Linux de nuestra empresa | Desplegar imágenes en los equipos del taller y en los de las sedes del cliente | SRI |
| Proxy de salida (Squid), opcional | Servidor Linux de nuestra empresa | Filtrar y registrar la navegación del taller por dominio | SRI, SAD |
| Windows Autopilot con Intune, solo documentado | En la nube de Microsoft | Lo que usaría el cliente para desplegar sus equipos si se pasara a Microsoft 365 | SRI |
| Copias de seguridad, solo documentado | Cada empresa | Cada sede guarda la copia de la otra, más una copia física cifrada | SAD, ASGBD |
| Intranet de reparaciones | Servidor Linux de nuestra empresa | Que el técnico consulte el historial de reparaciones de un cliente | IAW, ASGBD |

## Sitio web y base de datos

La idea: un solo sitio web en Python, con una única base de datos y dos zonas según el tipo de usuario: la pública, para clientes, y la de comerciales.

- **Comercial:** consulta el stock, crea pedidos y clientes, y ve el estado de pedidos e incidencias.
- **Cliente:** consulta el catálogo, se da de alta, modifica sus datos, hace pedidos y comunica incidencias.

La zona de comerciales es una web adaptada al móvil (no una app nativa), en una dirección propia (un subdominio o una ruta) que nginx solo permite desde la red de la VPN. La zona pública está abierta. En cada sede se publica con nginx para Windows y Python, dejado como servicio de Windows. El framework dependerá de lo que veamos en IAW. Las dos zonas comparten programa y base de datos, así que un fallo en una podría afectar a la otra: se separan con permisos bien definidos y, si la empresa crece, con servidores distintos. Conocemos el límite: nginx en Windows rinde menos que en Linux, aunque a este tamaño no se nota.

**Stock y precios.** El stock se guarda por producto y por almacén (sede). Los precios se leen siempre de la base de datos al hacer el pedido. Hay descuentos por campaña (con fechas), por tipo de producto o por tipo de cliente. No son acumulables: se aplica el mayor. El pedido guarda el precio y el descuento aplicados, para que cambiar una campaña no altere pedidos pasados. El diseño detallado se hará en ASGBD. El descuento de stock se hace en una sola operación que solo se cumple si hay existencias: si dos clientes compran el último producto a la vez, la base de datos deja pasar a uno y rechaza al otro.

**Base de datos con copia.** Hay una única base de datos principal, en la sede A, y una copia que recibe los cambios de forma continua por el túnel directo entre sedes, en la sede B. Todas las escrituras (pedidos, altas de clientes, entradas de stock) van a la principal, también las que se hacen desde la sede B. Es replicación y alta disponibilidad, no una base de datos distribuida en sentido estricto. Descartamos fragmentarla porque no aporta valor a una empresa de este tamaño, y así lo justificaremos en la memoria.

**Cómo se actualiza la copia.** Cada cambio en la base de datos principal (un pedido, una entrada de stock) se anota en un registro de transacciones (*redo*). Ese registro viaja por el túnel directo hasta la sede B, que repite los mismos cambios en el mismo orden. Con la réplica asíncrona, la principal confirma el cambio sin esperar a la copia, así que esta va unos instantes por detrás: es casi tiempo real, no tiempo real. Con la síncrona, la principal espera a la copia antes de confirmar: no se pierde nada, pero cada operación es más lenta y, si cae el túnel, la principal puede quedarse bloqueada. Elegimos asíncrona y documentamos el riesgo. Uptime Kuma puede vigilar el retraso de la réplica como extra.

> [!IMPORTANT]
> **MOTOR ELEGIDO: Oracle, con PostgreSQL 18 como plan B.** ASGBD nos ha dado el visto bueno para usar Oracle en las sedes. La réplica se hace con **Data Guard**: una copia en espera (*standby*) en modo asíncrono (*Maximum Performance*) y promoción manual con un comando. Como extra opcional, **Fast-Start Failover** promueve la copia de forma automática, decidido por un **observador** que corre en una tercera máquina: nuestro servidor del centro.
>
> **Pendiente de confirmar:** Data Guard solo está en la edición **Enterprise**; la gratuita (Free) no lo incluye. Hay que confirmar qué edición y licencia nos da el centro (por ejemplo, Oracle Academy) y si la versión funciona en Windows Server 2025. Si no es posible, el plan B es PostgreSQL 18, cuya réplica continua es gratuita y está probada en Windows Server 2025.

**Si cae la sede A:**

1. El técnico comprueba (en Uptime Kuma) que A no responde y que B sí.
2. Promueve B a principal con un comando, y la web de B pasa a atender los pedidos.
3. Los comerciales siguen entrando por el mismo nombre, que ahora apunta a B (con un TTL corto y un servidor DNS en cada sede).

Si montamos Fast-Start Failover (opcional), los pasos 1 y 2 los hace solo el observador, y la promoción manual queda como respaldo. Cuando A vuelve, se reincorpora como copia. Eso es manual y no entra en la demostración. Con réplica asíncrona pueden perderse los últimos segundos: un pedido a medias en el instante de la caída se pierde.

**Cómo lo haría una empresa real (para la memoria).** En una empresa de este tamaño es habitual un ERP con almacenes por sede, una base de datos central, a menudo en la nube, una línea de respaldo (4G o 5G) en cada sede y una tienda online separada que recibe el stock del ERP. Elegimos la réplica entre sedes porque nos permite demostrar ASGBD y SAD, y lo decimos con claridad.

Si la tienda online se nos va de las manos, la simplificamos (solo catálogo y alta de clientes) o la dejamos fuera.

## Cómo se autentican los usuarios en el sitio web

- **Comerciales de calle:** no trabajan en las sedes y no tienen cuenta del dominio. Entran con una cuenta propia de la web, guardada en la base de datos con la contraseña cifrada, y su rol les da acceso solo a su parte. Sus encargados de zona, que sí trabajan en el departamento comercial de la sede, tienen un rol propio para ver y gestionar a los comerciales de su zona. El acceso es siempre por HTTPS y solo a través de la VPN: el móvil del comercial lleva la app de WireGuard con un perfil personal, que se entrega con un código QR y se instala una sola vez. Cada comercial tiene su propia clave, así que se puede retirar el acceso a uno solo sin afectar al resto. Mejora opcional: un segundo factor con código temporal.
- **Clientes:** tienen cuenta propia en la web, con la contraseña guardada cifrada (hash) en la base de datos. Acceden por HTTPS, sin VPN, y solo ven sus datos y sus pedidos.
- **Permisos por rol:** cada vista (comercial o cliente) muestra solo lo que corresponde a su rol.
- **Decidido:** las cuentas de los comerciales viven en la base de datos, que se replica: si cae la sede A y B toma el relevo, siguen entrando sin hacer nada. Como mejora futura, la integración con Active Directory, que exigiría un único dominio con un controlador en cada sede.

**Alternativa sin VPN en el móvil (mejora futura).** Publicar la zona de comerciales en internet con HTTPS y un segundo factor. Es más cómodo para el comercial, pero deja esa zona visible en internet y depende por completo de la contraseña y del segundo factor. Elegimos la VPN por seguridad y por coherencia con el diseño.

## Herramientas de gestión y documentación

- **GitHub:** código del sitio web y documentación del proyecto.
- **OpenProject (edición Community, gratuita):** planificación, fases, hitos y reparto de tareas. No se expone a internet: solo es accesible por VPN.

**Estado actual de OpenProject.** Mientras el centro no ceda el servidor, funciona en una MV de Ubuntu Server con Docker Engine en el PC de Jordi, con la receta oficial de Docker Compose y la versión fijada (17.9.2). Los compañeros y los docentes entran por una VPN WireGuard a una dirección interna del túnel (`http://10.8.0.1:8080`), que se mantendrá igual en el servidor del centro: si todo va según lo previsto, el día de la mudanza no habrá que tocar nada en sus equipos. Ver los [manuales](manuales/).

## Alcance por prioridades

Primero la infraestructura y después el sitio web: si el sitio web luce pero la promoción de la copia no funciona, se va a notar.

1. **Imprescindible**: Active Directory en cada sede, DHCP y DNS, VPN entre sedes, firewall virtual con las cuatro redes de nuestra empresa, nginx con las dos webs y sus certificados, y base de datos con copia y promoción manual. Las copias de seguridad completas, los SAI y el despliegue de equipos por red solo se documentan.
2. **Deseable**: sitio web en Python con las dos zonas, y Uptime Kuma con un script que avise de que la copia del día se ha hecho.
3. **Opcional**: promoción automática con Fast-Start Failover y observador, una vez que todo lo demás funcione; tienda online completa con compra de prueba en la demostración, proxy de salida en el taller y un cortafuegos dedicado en las sedes si SAD lo pide.
