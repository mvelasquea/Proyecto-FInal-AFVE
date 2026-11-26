# Modelado Formal y Validación de un Sistema Crítico de Control y Cierre Automático de Efluentes Mineros (SIS-Efluentes)

Repositorio oficial del proyecto de la **Feria de Proyectos 2026** — Escuela Profesional de Ingeniería de Sistemas, Universidad Nacional de San Agustín de Arequipa (UNSA).

**Integrantes:**
- Anco Aymara Jean Pierre
- Cahuana Aguilar Josué Mathias Miguel
- Caracela Challco Anthony Jeffry
- Navarro Quispe Henry Alex
- Velasque Arcos Mikhail Gabino

**Docente asesor:** Msg. Maribel Molina Barriga

## Estructura del repositorio

| Archivo | Descripción |
| :--- | :--- |
| `SistemaSIS_Efluentes.vdmpp` | Especificación formal orientada a objetos en **VDM++** (`EstacionMonitoreo`, `ValvulaControl`, `SistemaSIS_Efluentes` y suite de 8 pruebas `TestSistemaSIS_Efluentes`). |
| `SistemaSIS_Efluentes.smv` | Modelo de transición de estados en **NuSMV** para verificación formal (*Model Checking*) con variables, guardas y 5 propiedades LTL (`G` y `F`). |
| `Diagrama_Estados_SIS_NuSMV.png` | Diagrama de transición de estados con variables y condiciones de guarda. |
| `Diagrama_Estados_SIS_NuSMV.puml` | Código fuente editable en PlantUML del diagrama de transición de estados. |
| `SCADA_SIS_Efluentes.html` | Simulador interactivo tipo SCADA (HTML autónomo) basado en las reglas del modelo formal. |

## Parte 1: Ejecución y validación en Overture (VDM++)

### Requisitos
- **Overture IDE**, **VS Code con la extensión VDM VSCode** (`overturetool.vdm-vscode`) o **VDMJ** con soporte para VDM++.

### Pasos de ejecución
1. Crear un nuevo proyecto VDM++ en **Overture** (o abrir la carpeta en VS Code con VDM VSCode) e importar `SistemaSIS_Efluentes.vdmpp`.
2. Verificar que el analizador estático no reporte errores de sintaxis ni de tipos y revisar las **18 Obligaciones de Prueba (POs)** en la vista *Proof Obligations*.
3. Abrir una sesión del intérprete y ejecutar los 6 casos funcionales positivos (`CP-01` a `CP-06`):

```vdmpp
p new TestSistemaSIS_Efluentes().ejecutarTodasLasPruebas()
```

Resultado esperado: `[true, true, true, true, true, true]`.

4. Ejecutar los 2 casos negativos por separado (ambos deben detener la ejecución con error en tiempo de ejecución, demostrando que el modelo rechaza estados inseguros):

```vdmpp
-- CP-07: Violacion de invariante Fail-Safe (As = 0.25 mg/L con valvula abierta -> Error 4060)
p new TestSistemaSIS_Efluentes().test07_NegativaInvarianteFailSafe()

-- CP-08: Violacion de precondicion de purga sin neutralizante (-> Error 4071)
p new TestSistemaSIS_Efluentes().test08_NegativaPrecondicionPurga()
```

## Parte 2: Validación de Model Checking en NuSMV

### Requisitos
- **NuSMV 2.6** (o superior) en macOS, Linux o Windows.

### Ejecución directa por línea de comandos

```bash
NuSMV SistemaSIS_Efluentes.smv
```

### Ejecución en modo interactivo (simulación y contraejemplo)

```bash
NuSMV -int SistemaSIS_Efluentes.smv
```

Dentro de la consola `NuSMV >`:

```text
read_model
flatten_hierarchy
encode_variables
build_model
check_ltlspec
pick_state -r
simulate -r -k 8
show_traces -v
check_ltlspec -p "G (valvula != CERRADA)"
quit
```

### Propiedades LTL verificadas

| ID | Tipo | Fórmula LTL en NuSMV | Significado | Resultado |
| :--- | :--- | :--- | :--- | :--- |
| **P1** | Seguridad (*Fail-Safe*) | `G (emergencia -> X (valvula != ABIERTA_DESCARGA))` | Ante infracción de LMP o falla de sensor, la válvula nunca queda abierta a descarga. | `true` |
| **P2** | Seguridad (Bloqueo total) | `G ((emergencia & volTanque = 4) -> X (valvula = CERRADA))` | Con emergencia y poza llena, ejecuta cierre hermético total. | `true` |
| **P3** | Seguridad (Capacidad) | `G (volTanque >= 0 & volTanque <= 4)` | El volumen de la poza nunca excede la capacidad máxima (`CAP_MAX = 4`). | `true` |
| **P4** | Seguridad (Purga) | `G ((volTanque > 0 & !neutralizanteListo) -> X (volTanque > 0))` | Nunca se purga el tanque sin neutralizante aplicado. | `true` |
| **P5** | Vivacidad (*Liveness*) | `G (purgaHabilitada -> F (valvula = ABIERTA_DESCARGA & volTanque = 0))` | Si se cumplen las condiciones de purga segura, el sistema vacía la poza y restablece descarga. | `true` |

## Simulador interactivo SCADA (Demo)

Para ejecutar la demostración visual sin instalar dependencias, abre `SCADA_SIS_Efluentes.html` directamente en cualquier navegador web (Chrome, Safari, Firefox o Edge). Permite simular lecturas de pH, cianuro, arsénico, SST y fallas de sensor, probar los escenarios `CP-01` a `CP-08` con un clic y exportar la bitácora OEFA a CSV.
