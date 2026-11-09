# Glosario

[Inicio](../README.md) · [Índice de la documentación](README.md)

Palabras y tecnicismos que aparecen en la documentación del proyecto, porque son muchos.

| Palabra | Qué significa |
| --- | --- |
| Active Directory (AD) | El servicio de Windows que guarda los usuarios, los grupos y sus permisos |
| BD | Base de datos |
| MV | Máquina virtual: un ordenador simulado dentro de otro |
| GPO | Reglas que se aplican a todos los equipos o usuarios del dominio a la vez |
| VPN y WireGuard | Un túnel privado entre dos sitios por internet; WireGuard es el programa que lo crea |
| RDP y SSH | Dos formas de controlar un ordenador a distancia: RDP, con escritorio; SSH, con comandos |
| SFTP | Transferencia de archivos por SSH, que podemos limitar a una sola carpeta |
| Proxy inverso | Un servidor web que recibe las peticiones de fuera y las reparte a los servicios de dentro |
| Certificado y CA | Un certificado demuestra que una web es quien dice ser; la CA es la autoridad que lo firma |
| Réplica | Una copia de la base de datos que recibe los cambios de forma continua |
| Promoción | Convertir la copia en la base de datos principal cuando la principal cae |
| Réplica en *streaming* y WAL | PostgreSQL anota cada cambio en un registro (WAL) y lo envía de forma continua a la copia, que lo repite |
| `pg_basebackup` | La herramienta que crea la copia inicial a partir de la base de datos principal |
| *Split-brain* | Cuando dos bases de datos se creen la principal a la vez y acumulan datos distintos |
| NSSM | Un programa que permite ejecutar nginx como servicio de Windows |
| NAT | Lo que permite que varios equipos salgan a internet con una sola dirección |
| CGNAT | Cuando el operador comparte una dirección pública entre muchos clientes y no se pueden abrir puertos |
| DNS dinámico | Un nombre que se actualiza solo cuando cambia nuestra IP pública |
| SAI | Una batería que da corriente unos minutos u horas cuando se va la luz |
| Copia incremental | Copia solo lo que ha cambiado desde la última copia |
| Docker | Un sistema para ejecutar programas aislados en «contenedores» |
| Sysprep y OVA | Sysprep prepara un Windows para clonarlo sin duplicar su identidad; OVA es el fichero con el que VirtualBox exporta e importa una MV completa |
| VBS | Seguridad basada en virtualización: Windows enciende su propio hipervisor, y VirtualBox va más lento |
| Evaluación y rearm | Versión de prueba de Windows (180 días el servidor, 90 el Windows 11); `slmgr /rearm` reinicia ese plazo un número limitado de veces |

---

[← IPE II](40-ipe-ii.md) · [Índice](README.md)
