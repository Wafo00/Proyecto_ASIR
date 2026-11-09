# Riesgos y limitaciones

[Inicio](../README.md) · [Índice de la documentación](README.md)

## Limitaciones conocidas y cómo se arreglarían

La empresa cliente está dando sus primeros pasos con una infraestructura propia: empezamos con lo que necesita hoy y con un presupuesto ajustado, y las mejoras se irán aplicando en todos los frentes conforme crezca. Somos conscientes de estas limitaciones y las explicaremos en la memoria y en la defensa, cada una con su solución futura.

| Limitación | Qué haríamos si la empresa crece |
| --- | --- |
| El servidor de cada sede hace de router, controlador de dominio, base de datos y web a la vez | Un cortafuegos dedicado por sede (OPNsense o pfSense) y el servidor solo en red interna |
| No hay DMZ | Una zona intermedia para la web, separada de la red de oficina |
| Sin protección frente a ataques de denegación de servicio de volumen | Protección del operador o un servicio antiDDoS delante de la web |
| Sin detección de intrusiones | IDS/IPS (por ejemplo Suricata) y registros centralizados |
| El concentrador de la VPN está en un único sitio | Un punto de entrada VPN por sede con IP fija; las sedes se conectarían entre sí y los comerciales, a la VPN de su propia sede |
| Web con nginx en Windows Server | Un servidor web dedicado, probablemente Linux |
| Base de datos con dos nodos y promoción manual | Promoción automática con una herramienta de alta disponibilidad (por ejemplo, Patroni, sobre Linux), o la base de datos en un servicio gestionado en la nube |
| Réplica asíncrona: pueden perderse los últimos segundos | Réplica síncrona, a costa de rendimiento |
| Nuestro servidor Linux no tiene alta disponibilidad | Un segundo nodo en Proxmox y copias fuera del centro |
| IP dinámica en la simulación (las sedes reales tendrían IP fija) | Nada: la empresa real tendría IP fija. En la simulación se usa DNS dinámico |
| Copias de seguridad solo documentadas | Un plan real con restauraciones periódicas y una copia fuera del sitio |
| Despliegue de equipos con un paquete de aprovisionamiento local | Windows Autopilot con Intune, al pasar a Microsoft 365 |

## Riesgos y planes B

El mayor riesgo es depender a la vez de tres sitios (el centro y dos casas) el día de la defensa.

| Riesgo | Plan B |
| --- | --- |
| El centro no nos cede el servidor, o lo comparte sin aislamiento suficiente | Montar todo en el equipo de Jordi (o uno más potente), con las sedes como redes internas de VirtualBox |
| Una conexión doméstica falla (IP dinámica, CGNAT, router, corte) | Comprobar la IP el día antes y el mismo día; si aun así falla, desplegar esa sede en el equipo de Jordi |
| La red del centro bloquea la VPN | Preguntar al docente de SRI por adelantado; el móvil como punto de acceso |
| El PC de la sede B no tiene recursos suficientes, permisos de administrador o virtualización, o tiene políticas que impiden desactivar la VBS | Medirlo y comprobarlo pronto; alternativa, un portátil del centro, y si tampoco sirve, el equipo de Jordi asume esa sede |
| El hipervisor de Windows (VBS, integridad de memoria, Windows Hello) se queda activo en un anfitrión: VirtualBox va lento y las instalaciones se cuelgan | Desactivarlo antes de crear las MV y comprobarlo en `msinfo32` |
| Se instala AD antes de cerrar el nombre del dominio: renombrarlo obliga a rehacer DNS y certificados | Los nombres de las empresas ya están decididos (Coral IT y Dextro Imports). No instalar AD hasta cerrar con ASO la pregunta del dominio |
| Se instala AD sobre la versión de evaluación: un controlador de dominio no puede convertirse a la versión con licencia | Riesgo asumido: el proyecto trabaja con evaluación y `rearm`. En una empresa real se instalaría con licencia |
| La evaluación de Windows caduca antes de la defensa (180 días el servidor, 90 el Windows 11) | Alarma diez días antes de la fecha que da `slmgr /xpr` en cada máquina, y `slmgr /rearm` con reinicio. Si ya no admite más, reinstalar desde la plantilla (OVA) antes de la fecha |
| Confusión entre las IP de los equipos reales y las de las máquinas virtuales | Seguir el [plan de direcciones](10-diseno-tecnico.md#plan-de-direcciones-y-nombres) desde el principio |
| La sede A vuelve sola tras la caída y las dos bases de datos se creen principales | La base de datos y la web de A no arrancan solas: las arranca el técnico cuando toca |
| Se estropea o se pierde una máquina virtual (por ejemplo, por un corte de luz) | Guardar la plantilla y cada sede exportadas como OVA, fuera del equipo anfitrión |
| Se cae algo el día de la defensa | Vídeos grabados de cada demostración clave, para proyectarlos y explicar en directo |
| El túnel directo A-B no se puede montar (CGNAT, puerto cerrado en casa) | La réplica depende de él: se comprueba el primer día y, si falla, se montan las dos sedes en el equipo de Jordi |

El guion de la demostración está en [La demostración](20-demostracion.md). Habrá que ensayarlo unos días antes y, una vez más, con el proyector.

---

[← La demostración](20-demostracion.md) · [Índice](README.md) · [IPE II →](40-ipe-ii.md)
