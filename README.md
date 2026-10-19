# Proyecto intermodular · 2º ASIR

Borrador v0.6 · 19 de octubre de 2026 · Pendiente de aprobar por el grupo

Montamos dos empresas ficticias y las conectamos: **la nuestra**, una pequeña empresa de servicios informáticos con taller, y **su primer cliente**, una empresa de importación y distribución con dos sedes. Nuestra empresa vive en un servidor del centro; cada sede del cliente, en un equipo distinto, unidas por una VPN. Lo defenderemos en 30-40 minutos, con una demostración en directo en la que cae una sede y la otra toma el relevo.

Este README es el resumen. El detalle está en la [documentación](#documentación).

## El proyecto en cinco puntos

- **Nuestra empresa** vive en un servidor Linux del centro (Proxmox): cuatro redes separadas por un firewall virtual, web con blog, intranet de reparaciones, base de datos Oracle, OpenProject y un panel de estado (Uptime Kuma).
- **La empresa cliente** tiene dos sedes. Cada una es un **Windows Server 2025** que lo hace todo (Active Directory, DNS, DHCP, archivos, base de datos y web) y un cliente **Windows 11** unido al dominio.
- **Las sedes se unen por VPN** (WireGuard) entre ellas y con el servidor del centro. La base de datos **Oracle** tiene la principal en la sede A y una copia continua en la B. Si cae la A, se promueve la B. PostgreSQL queda como plan B.
- **La defensa**: un portátil del centro hace de técnico de nuestra empresa, entra en las dos empresas y vemos caer la sede A y recuperarse. Antes, un bloque breve de IPE II sobre cómo nació nuestra empresa.
- **Lo que solo se documenta**: copias de seguridad completas, SAI, la tienda y el taller de nuestra empresa, y las medidas de seguridad más avanzadas.

## Esquema de la red

![Diagrama de red del proyecto](diagrama-red.png)

<details>
<summary>Ver o editar el diagrama como texto (Mermaid)</summary>

```mermaid
flowchart TB
    PORT["<b>Portátil de la demo</b><br/>Entra por VPN y RDP"]

    subgraph CENTRO["Servidor del centro (Proxmox)"]
        EMP["<b>Nuestra empresa</b><br/>Servidor Linux:<br/>web, BD y observador"]
        OPK["<b>OpenProject</b><br/>y Uptime Kuma<br/>en Docker"]
        FW["<b>Firewall y VPN</b><br/>Separa 4 redes<br/>y une las sedes"]
    end

    subgraph SEDEA["Sede A · PC de Jordi (dos máquinas virtuales)"]
        SA["<b>MV 1 · Windows Server 2025</b><br/>AD, DHCP, DNS, archivos y web<br/>BD principal y web activa"]
        CA["<b>MV 2 · Windows 11</b><br/>Unido al dominio"]
        CA -->|"entra con su usuario"| SA
    end

    subgraph SEDEB["Sede B · equipo de un compañero o portátil del centro (dos MV)"]
        SB["<b>MV 1 · Windows Server 2025</b><br/>AD, DHCP, DNS, archivos y web<br/>BD copia y web en reserva"]
        CB["<b>MV 2 · Windows 11</b><br/>Unido al dominio"]
        CB -->|"entra con su usuario"| SB
    end

    SA -->|"WireGuard"| FW
    SB -->|"WireGuard"| FW
    SA <-->|"túnel directo A-B (réplica de la BD)"| SB
    PORT -->|"VPN"| FW
    PORT -->|"RDP"| SA
    PORT -->|"RDP"| SB

    style FW fill:#ddf4ff,stroke:#0969da,stroke-width:2px
```

</details>

## Qué está decidido y qué no

| Estado | Tema | Detalle |
| --- | --- | --- |
| Decidido | Dónde va cada cosa | Nuestra empresa, en el servidor del centro. Sede A, en el PC de Jordi. Sede B, en el equipo de un compañero o en un portátil del centro |
| Decidido | Sistemas | Linux y Docker en nuestra empresa; Windows Server 2025 y Windows 11 en las sedes |
| Decidido | Base de datos | Oracle en las sedes (visto bueno de ASGBD), con réplica de A a B y promoción manual. PostgreSQL 18 como plan B |
| Decidido | VPN | WireGuard, con el concentrador en el servidor del centro y un túnel directo entre sedes para la réplica |
| Decidido | Webs | nginx con certificados de una CA propia (openssl). Web del cliente en Python |
| Decidido | Herramientas | GitHub para la documentación y OpenProject para la planificación |
| Pendiente de confirmar | Edición de Oracle | La réplica con promoción (Data Guard) solo está en la edición Enterprise. Hay que confirmar qué edición y licencia nos da el centro y si funciona en Windows Server 2025 |
| Pendiente | Servidor del centro | El docente está de acuerdo; previsto en un par de semanas |
| Por decidir | Nombres de las dos empresas | Entre todos. Bloquean el dominio de AD, los nombres DNS y los certificados |
| Por decidir | Restauración de copia en la defensa | Si se demuestra y cómo, por el tiempo limitado |
| Por decidir | Motivo de la caída de la sede A | El gato que pulsa el botón de apagado o similar, a ser posible con gracia |

## La defensa en 40 minutos

| Bloque | Minutos |
| --- | --- |
| IPE II: cómo nació nuestra empresa (preliminar) | 8-10 |
| Diseño de la infraestructura: diagrama y decisiones | 6-8 |
| Demo: el técnico entra en las dos empresas | 6-8 |
| Demo: cae la sede A y la B toma el relevo | 5-6 |
| Limitaciones, mejoras y cierre | 3-4 |
| Margen para imprevistos y preguntas | 4-5 |

Guion completo en [La demostración](docs/demostracion.md).

## Calendario

| Hito | Fecha prevista |
| --- | --- |
| Decisiones previas cerradas | Principios de noviembre de 2026 |
| Sedes A y B con Active Directory | Finales de noviembre |
| Túnel entre sedes y réplica de la base de datos | Diciembre |
| Webs con HTTPS y nuestra empresa completa en el centro | Enero de 2027 |
| Demo completa probada | Principios de febrero, antes de las prácticas |
| Prácticas en empresa (sin trabajo en el proyecto) | Mediados de febrero a mediados de abril |
| Memoria, ensayos y defensa | Finales de mayo (fecha por confirmar) |

El planning detallado, con fases, hitos y tareas, está en OpenProject.

## Documentación

| Documento | Qué contiene |
| --- | --- |
| [Diseño técnico](docs/diseno-tecnico.md) | Las dos empresas, la infraestructura, los servicios, la red, la web, la base de datos y la autenticación |
| [La demostración](docs/demostracion.md) | Guion de la defensa, acceso remoto y comprobaciones del día antes |
| [Riesgos y limitaciones](docs/riesgos-y-limitaciones.md) | Qué puede fallar, el plan B de cada cosa y qué haríamos si la empresa creciera |
| [IPE II](docs/ipe-ii.md) | La parte de empresa: cómo nació nuestra empresa (preliminar) |
| [Seguimiento](docs/seguimiento.md) | Preguntas pendientes, contenidos por ver, notas e historial de versiones |
| [Glosario](docs/glosario.md) | Los tecnicismos del proyecto, explicados |
| [Manuales](docs/manuales/) | Procedimientos paso a paso: [conectarse a OpenProject por VPN](docs/manuales/conexion-vpn-openproject.md), [montaje de esa VPN](docs/manuales/montaje-vpn-openproject.md) y [acceso al servidor del centro](docs/manuales/acceso-servidor-centro.md) (provisional) |

## Estado actual

- **OpenProject** funciona de forma provisional en una máquina virtual del PC de Jordi, con fases e hitos del planning. Se migrará al servidor del centro cuando esté disponible.
- **VPN WireGuard** montada y probada desde fuera de casa; los compañeros y los docentes entran a OpenProject con su propio perfil.
- **Sede A**: Windows Server 2025 instalado y actualizado.
