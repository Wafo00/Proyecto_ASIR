# Glosario

[← Volver al README](../README.md)

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
| Data Guard | La réplica de Oracle (edición Enterprise): mantiene una copia en espera que recibe los cambios de la principal |
| Fast-Start Failover y observador | La promoción automática de Data Guard; el observador es un tercer proceso, en otra máquina, que decide cuándo promover la copia |
| Testigo | Un tercer nodo que no guarda datos y desempata sobre quién ha caído (en Oracle, el observador) |
| NAT | Lo que permite que varios equipos salgan a internet con una sola dirección |
| CGNAT | Cuando el operador comparte una dirección pública entre muchos clientes y no se pueden abrir puertos |
| DNS dinámico | Un nombre que se actualiza solo cuando cambia nuestra IP pública |
| SAI | Una batería que da corriente unos minutos u horas cuando se va la luz |
| Copia incremental | Copia solo lo que ha cambiado desde la última copia |
| Docker | Un sistema para ejecutar programas aislados en «contenedores» |
| Sysprep y OVA | Sysprep prepara un Windows para clonarlo sin duplicar su identidad; OVA es el fichero con el que VirtualBox exporta e importa una MV completa |
| VBS | Seguridad basada en virtualización: Windows enciende su propio hipervisor, y VirtualBox va más lento |
| Evaluación y rearm | Windows Server de prueba, 180 días; `slmgr /rearm` reinicia ese plazo un número limitado de veces |
