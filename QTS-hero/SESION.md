# Registro de sesión de desarrollo — Template QNAP QTS hero Zabbix 7.2

Documento de recuperación de memoria para Claude Code.
**Fechas:** 22-23 febrero 2026
**Repo:** `rubencastello1990/zabbix-qnap-qts-hero-snmp`
**Fichero principal:** `QTS-hero/template_qnap_qts_hero_zabbix72.yaml`

---

## Contexto del proyecto

Se creó desde cero un **template Zabbix 7.2 en YAML** para monitorización SNMP completa de NAS QNAP con firmware **QuTS hero (h5.x+)**.

- MIB enterprise: `1.3.6.1.4.1.55062.2.x` (sub-nodo 2 = QTS hero; el sub-nodo 1 es QTS clásico)
- MIB de referencia local: `/home/ciberstorm/NAS.mib`
- Documento de mapeo OIDs: `QTS-hero/MAPPING.md`
- El template base de referencia es uno comunitario para Zabbix 6.x que usaba OIDs legacy incompatibles

---

## Historial completo de cambios (todos los commits)

### Commit `de757a4` — Creación inicial
**feat: add QNAP QTS hero / QuTS hero template for Zabbix 7.2**

Template base creado con:
- 26 ítems estáticos (sistema, CPU, memoria, UPS, caché, contadores)
- 10 LLD discovery rules
- 26 ítems prototipo
- 10 triggers escalares, 18 trigger prototipos
- 10 value maps

---

### Commit `9b4822d` — Fix formato YAML
**fix: remove `date` field — not valid for Zabbix 7.2 import**

Zabbix 7.2 rechaza el campo `date` en el YAML exportado. Eliminado.

---

### Commit `b98f562` — Fix estructura YAML
**fix: rename `template_groups` to `groups` inside template definition**

En Zabbix 7.2, dentro del bloque `templates[].`, la clave correcta es `groups:`, no `template_groups:`.

---

### Commit `bc26df9` — Fix posición de triggers
**fix: move triggers inline into items (Zabbix 7.2 format)**

En Zabbix 7.2 YAML, los triggers de ítems escalares deben ir anidados dentro del ítem correspondiente (campo `triggers:`), no en una sección `triggers:` de nivel superior del template.

---

### Commit `9457a36` — Fix formato UUIDs
**fix: remove hyphens from UUIDs (Zabbix requires 32 hex chars)**

Zabbix requiere UUIDs como 32 caracteres hex sin guiones. Se eliminaron los guiones de todos los UUIDs (`xxxxxxxx-xxxx-xxxx` → `xxxxxxxxxxxxxxxx`).

---

### Commit `bf4e687` — Fix UUIDs secuenciales
**fix: replace sequential UUIDs with proper random uuid4 values**

Los UUIDs iniciales eran secuenciales (`00000001`, `00000002`...), lo que puede causar colisiones con otros templates. Se sustituyeron por `uuid.uuid4().hex` reales.

---

### Commit `75ef032` — Reescritura crítica
**fix: complete rewrite with critical corrections**

Correcciones múltiples tras validación exhaustiva del YAML frente al esquema Zabbix 7.2.

---

### Commit `cb2d34f` — Trigger prototypes completos
**feat: add trigger prototypes to all LLD discovery rules**

Se añadieron trigger prototipos a todas las reglas LLD donde faltaban:
- Disk: SMART Error (DISASTER), SMART Warning (HIGH), Temp HIGH, Temp WARN
- Fan: ventilador parado (HIGH)
- RAID: Degraded/Dead (DISASTER), Rebuilding/Sync (WARNING)
- Pool: Not Ready (HIGH), espacio crítico (AVERAGE), espacio bajo (WARNING)
- Volume: Not Ready (DISASTER), espacio crítico (AVERAGE), espacio bajo (WARNING)
- LUN: Not Enabled/Ready (HIGH)
- Target iSCSI: Offline (HIGH)
- Network IF: Link down (AVERAGE)
- Enclosure: temperatura alta (WARNING)
- mSATA: SMART error (HIGH)

---

### Commit `660b30b` — 4 bugs críticos
**fix: 4 bugs críticos + 1 warning + 1 simplificación**

Bugs corregidos:
1. OID de discovery para volúmenes/carpetas compartidas incorrecto → corregido a `.55062.2.10.9.1.1`
2. Macros LLD mal referenciadas en filtros de discovery
3. Fórmula de % libre en volúmenes con división por cero potencial → añadida protección
4. Varios `value_type` incorrectos en ítems numéricos

---

### Commit `dba2491` — Falso positivo disco Standby + delays LLD
**fix: falso positivo disk Standby + escalonar delays discovery rules**

**Problema:** El trigger de disco "Not Operational" disparaba en discos en estado Standby (estado legítimo cuando el disco está en reposo para ahorro energético).

**Solución:** Filtrado del estado Standby en la expresión del trigger, solo disparar en estados genuinamente problemáticos.

**Además:** Se escalonaron los delays de las 10 reglas LLD para evitar picos de carga SNMP simultánea en el primer ciclo de discovery tras importar el template.

---

### Commit `3eefd3e` — Optimización carga SNMP + slot en nombres
**perf: reducir carga SNMP + añadir slot a nombres de disco**

- Se añadió el número de slot del NAS al nombre de display de los discos LLD para facilitar identificación física (`Disk [Slot 3]: SMART Status` en vez de `Disk [2]: SMART Status`)
- Se revisaron y aumentaron los delays de ítems de baja prioridad para reducir la carga SNMP sobre el NAS

---

### Commit `49dffd5` — Modelo/serie de disco como ítems + triggers granulares
**feat: disk model/serial como ítems + triggers Error/Warning granulares**

- Añadidos ítems prototipo para modelo y número de serie de cada disco
- Separados los triggers de SMART en dos niveles: Error (DISASTER) y Warning (HIGH) en vez de un único trigger combinado
- Esto permite mayor granularidad en la gestión de alertas por disco

---

### Commit `d548e28` — Fix UUIDs inválidos en disco
**fix: corregir UUIDs inválidos en items/triggers de disco**

Los nuevos ítems de modelo/serie y los triggers granulares tenían UUIDs mal formados (longitud incorrecta). Corregidos con `uuid4().hex`.

---

### Commit `521b256` — Fix: añadir uuid a dependencies
**fix: añadir uuid a todas las dependencies de triggers**

Zabbix 7.2 requiere campo `uuid` en las referencias de `dependencies` entre triggers. Añadido a todas las entradas de dependencias que carecían de él.

---

### Commit `60250fe` — Fix: quitar uuid de dependencies en triggers inline
**fix: quitar uuid de dependencies en triggers inline de items escalares**

Zabbix 7.2 no acepta `uuid` en el campo `dependencies` dentro de triggers anidados en ítems. Eliminado donde aplicaba.

---

### Commits `9be0150` + `4eb7284` — Fix dependencies incompatibles
**fix: eliminar uuid + eliminar dependencies de trigger 'not Ready'**

El sistema de `dependencies` entre trigger prototipos LLD no funciona correctamente en Zabbix 7.2 cuando se exporta/importa YAML. Se eliminaron todas las dependencies problemáticas del trigger `Pool: Status is not Ready`, que causaban errores de importación.

---

### Commit `0f0fa6e` — Fix falso positivo disco 'Good'
**fix(disk): add 'Good' as valid operational state to prevent false positives**

**Problema:** El trigger de disco "Not Operational" disparaba también cuando el estado era `Good`, ya que `Good` no estaba explícitamente excluido como estado válido.

**Causa raíz:** La expresión del trigger usaba lógica negativa incompleta — solo excluía algunos estados buenos conocidos, pero `Good` se coló como estado no excluido.

**Solución:** Añadido `Good` a la lista de estados operacionales válidos en la expresión del trigger.

**Commit relevante:** `0f0fa6e`

---

### Commit `ec7871b` — Fix trigger SNMP No response (PRINCIPAL)
**fix(trigger): replace nodata() with zabbix[host,snmp,available] INTERNAL item**

**Problema:** El trigger `SNMP: No response from host` (uuid `2932cf4f9ade48a297efa38c3ead3c21`) generaba falsos positivos continuos.

**Causa raíz — error de diseño:**

| | Expresión | Problema |
|---|---|---|
| Expresión incorrecta | `nodata(/QNAP QTS hero by SNMP/system.uptime,{$SNMP.TIMEOUT})=1` | Falsos positivos en reinicio servidor, import del template, cambio de intervalo, delays del poller |
| Expresión correcta | `max(/QNAP QTS hero by SNMP/zabbix[host,snmp,available],{$SNMP.TIMEOUT})=0` | Item INTERNAL que Zabbix actualiza directamente con el estado real del poller SNMP |

**Cambios implementados:**
1. Añadido nuevo ítem INTERNAL `zabbix[host,snmp,available]` (UUID `27a5d53427944401bffc8f38d2b0c781`, delay=1m, history=7d, tag: component=health)
2. Movido el trigger `2932cf4f9ade48a297efa38c3ead3c21` del ítem `system.uptime` al nuevo ítem INTERNAL
3. Expresión actualizada: `nodata(system.uptime,...)=1` → `max(zabbix[host,snmp,available],...)=0`
4. El ítem interno refleja 0=no disponible, 1=disponible, 2=desconocido — actualizado directamente por el poller, sin depender del historial de ningún OID

---

### Commit `e8da5f4` — Fix CALCULATED items 'does not exist' (FINAL)
**fix(calculated): resolve 'item does not exist' on Memory and Pool free% items**

**Problema:** Dos ítems CALCULATED fallaban con error `Cannot evaluate function: item does not exist`:
- `Memory: Utilization` (key: `vm.memory.util`)
- `Pool [N]: Free space %` (key: `storagepool.free.pct[{#SNMPINDEX}]`)

**Causa raíz — race condition de delay:**

El error `item does not exist` en ítems CALCULATED de Zabbix NO significa que el ítem no esté en la BD — significa que el ítem referenciado no tiene ningún dato en el histórico en el momento de la evaluación. Zabbix lo reporta con ese mensaje confuso.

Para `Memory: Utilization`:
- `vm.memory.total` tenía delay=**1h** (RAM total no cambia, se polleaba poco)
- `vm.memory.util` tenía delay=**1m** (evaluación cada minuto)
- En los primeros ~60 minutos tras importar, `vm.memory.util` evaluaba la fórmula pero `vm.memory.total` no había hecho su primer poll todavía → error `does not exist` en cada evaluación

Para `Pool [N]: Free space %`:
- `storagepool.capacity` y `storagepool.free` tienen delay=**15m**
- `storagepool.free.pct` tenía delay=**15m** también
- LLD crea todos los ítems prototipo simultáneamente; el CALCULATED puede evaluarse en el mismo ciclo que los SNMP hacen su primer poll → race condition

**Fixes:**
- `vm.memory.total`: delay `1h` → `3m` (RAM total es estática, overhead mínimo, elimina el timing issue)
- `storagepool.free.pct`: delay `15m` → `16m` (1 minuto extra garantiza que los ítems SNMP fuente ya tienen su primer dato almacenado)

---

## Estado final del template (post-sesión)

### Estructura de ficheros
```
QTS-hero/
├── template_qnap_qts_hero_zabbix72.yaml  ← Template Zabbix (importar)
├── README.md                              ← Documentación de uso
├── MAPPING.md                             ← Mapeo completo de OIDs (referencia técnica)
└── SESION.md                              ← Este fichero (registro de sesión)
```

### Ítems clave y sus UUIDs

| Ítem | UUID | Key | Tipo | Delay |
|------|------|-----|------|-------|
| SNMP: Availability | `27a5d53427944401bffc8f38d2b0c781` | `zabbix[host,snmp,available]` | INTERNAL | 1m |
| Memory: Total | `8a62925d9a9f4a7d8973e06a96e825d6` | `vm.memory.total` | SNMP_AGENT | 3m |
| Memory: Used | `0b0d4604bdc345a8b79ff8537b2ef068` | `vm.memory.used` | SNMP_AGENT | 3m |
| Memory: Utilization | `d4e2f77c40654f65869cd5701a211c3f` | `vm.memory.util` | CALCULATED | 1m |
| System: Uptime | `a16670219c98428f9c55d8fe2334eff2` | `system.uptime` | SNMP_AGENT | 1m |

### Trigger crítico corregido

| UUID | Nombre | Expresión actual (correcta) |
|------|--------|----------------------------|
| `2932cf4f9ade48a297efa38c3ead3c21` | SNMP: No response from host | `max(/QNAP QTS hero by SNMP/zabbix[host,snmp,available],{$SNMP.TIMEOUT})=0` |

### Pool Free Space % — delay final

| Ítem prototipo | Key | Delay |
|----------------|-----|-------|
| Pool: Capacity | `storagepool.capacity[{#SNMPINDEX}]` | 15m |
| Pool: Free space | `storagepool.free[{#SNMPINDEX}]` | 15m |
| Pool: Free space (%) | `storagepool.free.pct[{#SNMPINDEX}]` | 16m ← un minuto extra |

---

## Notas técnicas importantes para futuras sesiones

### OIDs de memoria — unidad
La MIB (`NAS.mib`) define `systemTotalMem` y `systemUsedMemory` como `Counter64` sin unidad explícita. El MAPPING.md indica que devuelven **MB**. El cálculo de % es correcto independientemente de la unidad (ratio es igual en MB o bytes).

### OIDs de pool — unidad
La MIB define `storagepoolCapacity` y `storagepoolFreeSize` como `Counter64` con descripción explícita "**in byte**". Por tanto los ítems NO necesitan MULTIPLIER de conversión.

### SNMP Availability — ítem INTERNAL
El ítem `zabbix[host,snmp,available]` es un ítem INTERNO de Zabbix (type: INTERNAL). No consulta ningún OID. Zabbix lo actualiza automáticamente con el resultado real del poller SNMP:
- `0` = no disponible (trigger DISASTER se activa si `max()=0` durante `{$SNMP.TIMEOUT}`)
- `1` = disponible
- `2` = desconocido

### Triggers anidados vs. nivel raíz
En Zabbix 7.2 YAML, los triggers DEBEN ir anidados dentro del ítem al que pertenecen (campo `triggers:`). NO existe sección `triggers:` de nivel raíz en el template (a diferencia de Zabbix 6.x).

### Dependencies entre trigger prototipos LLD
Las `dependencies` entre trigger prototipos NO funcionan correctamente en el formato YAML de Zabbix 7.2 para importación. Se eliminaron todas. Si se necesitan en el futuro, configurarlas manualmente desde la UI de Zabbix tras la importación.

### Disco: estado Standby es válido
El trigger de disco "Not Operational" DEBE excluir el estado `Standby`. Es un estado operacional legítimo (disco en reposo). Si no se excluye, genera falsos positivos en NAS con gestión energética de discos activada.

### Validación YAML
Siempre validar antes de hacer push:
```bash
python3 -c "import yaml; yaml.safe_load(open('template_qnap_qts_hero_zabbix72.yaml')); print('OK')"
```

---

## Comandos útiles de referencia

```bash
# Verificar SNMP conectividad y OIDs clave
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.13.0  # Memory Total
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.16.0  # Memory Used
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.7    # Pool table
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.2    # Disk table

# Generar UUID válido para Zabbix
python3 -c "import uuid; print(uuid.uuid4().hex)"

# Validar YAML
python3 -c "import yaml; yaml.safe_load(open('QTS-hero/template_qnap_qts_hero_zabbix72.yaml')); print('OK')"
```
