# Réplica de las sedes

[← Índice de la documentación](../README.md)

Los pasos exactos, escuetos y para copiar y pegar con los que se monta cada sede de Dextro Imports. La sede A es la referencia: lo que se hace allí se apunta aquí y la sede B (o una sede A rota que haya que rehacer) lo repite.

> [!NOTE]
> **En preparación.** Estos documentos se escriben a medida que se monta la sede A, para que cada comando esté probado. Aún no hay pasos publicados.

## Qué habrá aquí

| Documento | Qué cubre |
| --- | --- |
| `servidor-sede.md` | Windows Server 2025 de la sede: sistema, red, servicios, nginx y PostgreSQL |
| `cliente-sede.md` | Windows 11 de la sede: sistema, unión al dominio y herramientas de consulta |

Cada documento tiene dos partes:

1. **De cero hasta antes de clonar.** Es común a las dos sedes y termina con la plantilla generalizada (Sysprep) y exportada como OVA.
2. **Después de clonar, hasta el final.** Lo propio de cada sede, con una tabla de valores para A y para B (nombres, direcciones, claves).

## Normas de estos documentos

- Una línea de explicación por paso y los comandos tal cual, para copiar y pegar.
- Nunca incluyen contraseñas ni claves. Donde haga falta, dicen «elige una contraseña» y la guardan quien monta la sede.
- Los nombres y direcciones salen del [plan de direcciones](../10-diseno-tecnico.md#plan-de-direcciones-y-nombres).
- Al final de cada parte, una comprobación para saber que ha ido bien y la copia (OVA) de la máquina.
