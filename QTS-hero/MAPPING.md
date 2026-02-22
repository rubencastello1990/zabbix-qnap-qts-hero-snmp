# OID Mapping: Template base Zabbix 6.x → QTS hero Zabbix 7.2

Documento de referencia técnica para el proceso de migración de OIDs del template comunitario
de Zabbix 6.x hacia la MIB QTS-MIB (`/home/ciberstorm/NAS.mib`) del firmware QuTS hero.

## Contexto de la migración

El template comunitario base (para Zabbix 6.x, firmware QTS 4.5.2+) utiliza dos bases OID
incompatibles con QuTS hero:

```
1.3.6.1.4.1.24681     → Enterprise QNAP legacy (QTS 4.x). NO existe en NAS.mib de QTS hero.
1.3.6.1.4.1.55062.1.x → Nodo "qts" (sub-nodo 1). QTS hero usa sub-nodo 2.
1.3.6.1.4.1.55062.2.x → Nodo "qutshero" (sub-nodo 2). BASE CORRECTA para QTS hero.
```

La MIB local (`NAS.mib`) define el módulo `QTS-MIB` con enterprise base `55062.2`.

---

## 1. Ítems estáticos

### Cambio de sub-nodo: .1. → .2. (directo)

| Ítem | OID Template Base (6.x) | OID QTS hero (7.2) | Estado |
|------|------------------------|---------------------|--------|
| Cache: Mode | `55062.1.10.17.0` | `55062.2.10.17.0` | ✅ Cambia (.1.→.2.) |
| Cache: RAID Type | `55062.1.10.16.0` | `55062.2.10.16.0` | ✅ Cambia (.1.→.2.) |
| Cache: Read hit rate | `55062.1.10.12.0` | `55062.2.10.12.0` | ✅ Cambia (.1.→.2.) |
| Cache: Type | `55062.1.10.15.0` | `55062.2.10.15.0` | ✅ Cambia (.1.→.2.) |
| Cache: Write hit rate | `55062.1.10.13.0` | `55062.2.10.13.0` | ✅ Cambia (.1.→.2.) |
| System: Firmware update | `55062.1.12.7.0` | `55062.2.12.7.0` | ✅ Cambia (.1.→.2.) |
| System: Firmware version | `55062.1.12.6.0` | `55062.2.12.6.0` | ✅ Cambia (.1.→.2.) |
| System: Hostname | `55062.1.12.4.0` | `55062.2.12.4.0` | ✅ Cambia (.1.→.2.) |
| System: Serial no. | `55062.1.12.5.0` | `55062.2.12.5.0` | ✅ Cambia (.1.→.2.) |

### Cambio de OID enterprise (24681 → 55062.2)

| Ítem | OID Template Base (6.x) | OID QTS hero (7.2) | Estado |
|------|------------------------|---------------------|--------|
| Cache: Available % | `24681.1.4.1.1.1.3.2.0` | `55062.2.10.11.0` | ✅ OID diferente |
| CPU: Temperature | `24681.1.4.1.1.1.1.4.2.0` | `55062.2.12.10.0` | ✅ OID diferente |
| System: Temperature | `24681.1.4.1.1.1.1.1.2.1.7.1` | `55062.2.12.11.0` | ✅ OID diferente |

### Sin cambio

| Ítem | OID | Estado |
|------|-----|--------|
| System: Uptime | `1.3.6.1.2.1.1.3.0` | ✅ Sin cambio (MIB-II estándar) |
| SNMP availability | internal | ✅ Sin cambio |

### OIDs no disponibles en NAS.mib

| Ítem | OID Template Base | Motivo | Estado |
|------|-------------------|--------|--------|
| Disk: Overall IOPS | `24681.1.4.1.11.5.6.2.1.3.1` | OID legacy 24681, no documentado en NAS.mib | ❌ NO DISPONIBLE |
| Disk: Overall latency | `24681.1.4.1.11.5.6.2.1.4.1` | OID legacy 24681, no documentado en NAS.mib | ❌ NO DISPONIBLE |

### Ítems nuevos (no existen en template base)

Estos ítems son funcionalidades adicionales de la MIB QTS hero que el template base no tiene:

| Ítem | OID | Tipo | Descripción |
|------|-----|------|-------------|
| System: Model | `55062.2.12.3.0` | Escalar string | Modelo del NAS |
| System: CPU Usage % | `55062.2.12.12.0` | Escalar numérico | Uso CPU total |
| Memory: Total | `55062.2.12.13.0` | Escalar numérico (MB) | RAM total |
| Memory: Used | `55062.2.12.16.0` | Escalar numérico (MB) | RAM usada |
| Memory: Utilization | Calculado | % | Used/Total*100 |
| System: Power Status | `55062.2.12.19.0` | Escalar enum | OK/Failed |
| System: UPS status code | `55062.2.12.20.0` | Escalar numérico | Código UPS interno |
| UPS: Status text | `55062.2.13.1.0` | Escalar string | Estado UPS |
| UPS: Battery charge | `55062.2.13.7.0` | Escalar numérico (%) | Batería UPS |
| UPS: Load | `55062.2.13.6.0` | Escalar numérico (%) | Carga UPS |
| Storage: Disk count | `55062.2.10.1.0` | Escalar numérico | N.º discos |
| Storage: RAID count | `55062.2.10.4.0` | Escalar numérico | N.º RAIDs |
| Storage: Pool count | `55062.2.10.6.0` | Escalar numérico | N.º pools |

---

## 2. LLD Rules

### Cambio de sub-nodo: .1. → .2. (directo)

| LLD Rule | OID Template Base (6.x) | OID QTS hero (7.2) | Estado |
|----------|------------------------|---------------------|--------|
| Disk Discovery | `55062.1.10.2.1.1` | `55062.2.10.2.1.1` | ✅ Cambia (.1.→.2.) |
| FAN Discovery | `55062.1.12.9.1.1` | `55062.2.12.9.1.1` | ✅ Cambia (.1.→.2.) |
| LUN Discovery | `55062.1.10.20.1.1` | `55062.2.10.20.1.1` | ✅ Cambia (.1.→.2.) |
| LUN Target Discovery | `55062.1.10.22.1.1` | `55062.2.10.22.1.1` | ✅ Cambia (.1.→.2.) |
| Pool Discovery | `55062.1.10.7.1.1` | `55062.2.10.7.1.1` | ✅ Cambia (.1.→.2.) |
| RAID Discovery | `55062.1.10.5.1.1` | `55062.2.10.5.1.1` | ✅ Cambia (.1.→.2.) |

### Cambio de OID enterprise o tabla

| LLD Rule | OID Template Base (6.x) | OID QTS hero (7.2) | Estado |
|----------|------------------------|---------------------|--------|
| Network IF Discovery | `24681.1.2.9.1.1` (propietario) | IF-MIB `1.3.6.1.2.1.2.2.1.1` (estándar) | ✅ Cambia (usar IF-MIB) |
| Volume Discovery | `24681.1.4.1.1.1.2.3.2.1.1` | `55062.2.10.9.1.1` (sharedFolderTable) | ✅ OID diferente |

### OIDs no disponibles en NAS.mib

| LLD Rule | OID Template Base | Motivo | Estado |
|----------|-------------------|--------|--------|
| CPU Core Discovery | `24681.1.4.1.1.1.1.4.3.1.1` | Solo CPU total en NAS.mib; no hay tabla de cores | ❌ NO DISPONIBLE |

### LLD Rules nuevas (no existen en template base)

Funcionalidades de QTS hero MIB sin equivalente en el template base:

| LLD Rule | OID base | Descripción |
|----------|----------|-------------|
| Enclosure Discovery | `55062.2.10.34` | Recintos de expansión QNAP (temperatura) |
| mSATA Cache Disk Discovery | `55062.2.10.32` | Discos mSATA de caché (SMART, temp, capacidad) |

---

## 3. Columnas de tablas SNMP

### diskTable (`.55062.2.10.2.1`)

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.1` | diskIndex | `{#DISKINDEX}` | Índice de la tabla |
| `.2` | diskID | `{#DISKID}` | Identificador del disco |
| `.3` | diskManufacturer | `{#DISKMANUFACTURER}` | Fabricante |
| `.4` | diskModel | `{#DISKMODEL}` | Modelo |
| `.5` | diskSerialNumber | `{#DISKSERIALNUMBER}` | Número de serie |
| `.6` | diskType | — | Tipo de disco |
| `.7` | diskSMARTStatus | — | Estado SMART (ítem prototipo) |
| `.8` | diskTemperature | — | Temperatura (ítem prototipo) |
| `.9` | diskCapacity | — | Capacidad en MB (ítem prototipo) |

### raidTable (`.55062.2.10.5.1`)

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.1` | raidIndex | `{#RAIDINDEX}` | Índice de la tabla |
| `.2` | raidID | `{#RAIDID}` | Identificador RAID |
| `.3` | raidName | `{#RAIDNAME}` | Nombre del RAID |
| `.4` | raidStatus | — | Estado (ítem prototipo, valor numérico traducido por JS) |
| `.5` | raidCapacity | — | Capacidad en MB (ítem prototipo) |
| `.7` | raidLevel | — | Nivel RAID string (ítem prototipo) |

### storagepoolTable (`.55062.2.10.7.1`)

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.1` | storagepoolIndex | `{#STORAGEPOOLINDEX}` | Índice de la tabla |
| `.2` | storagepoolID | `{#STORAGEPOOLID}` | Identificador del pool |
| `.3` | storagepoolCapacity | — | Capacidad en MB (ítem prototipo) |
| `.4` | storagepoolFreeSize | — | Espacio libre en MB (ítem prototipo) |
| `.5` | storagepoolStatus | — | Estado (ítem prototipo, valuemap) |

### sharedFolderTable (`.55062.2.10.9.1`)

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.2` | sharedFolderName | `{#SHAREDFOLDERNAME}` | Nombre del volumen |
| `.3` | sharedFolderCapacity | — | Capacidad en MB (ítem prototipo) |
| `.4` | sharedFolderFreeSize | — | Espacio libre en MB (ítem prototipo) |
| `.5` | sharedFolderStatus | `{#SHAREDFOLDERSTATUS}` / ítem prototipo | Estado textual |

### lunTable (`.55062.2.10.20.1`)

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.1` | lunIndex | `{#LUNINDEX}` | Índice de la tabla |
| `.3` | lunCapacity | — | Capacidad en MB (ítem prototipo) |
| `.5` | lunStatus | — | Estado textual (ítem prototipo) |
| `.6` | lunName | `{#LUNNAME}` | Nombre del LUN |
| `.8` | lunMapped | — | Estado de mapeo (ítem prototipo, valuemap) |

### targetTable (`.55062.2.10.22.1`)

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.1` | targetIndex | `{#TARGETINDEX}` | Índice de la tabla |
| `.3` | targetName | `{#TARGETNAME}` | Nombre del target |
| `.4` | targetIQN | `{#TARGETIQN}` | IQN del target |
| `.6` | targetStatus | — | Estado (ítem prototipo, valuemap) |

### sysFanTable (`.55062.2.12.9.1`)

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.1` | sysFanIndex | `{#SYSFANINDEX}` | Índice de la tabla |
| `.2` | sysFanDescr | `{#SYSFANDESCR}` | Descripción/nombre del ventilador |
| `.3` | sysFanSpeed | — | Velocidad en RPM (ítem prototipo) |

### enclosureTable (`.55062.2.10.34.1`) — NUEVO en QTS hero

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.1` | enclosureIndex | `{#ENCLOSUREINDEX}` | Índice de la tabla |
| `.3` | enclosureModel | `{#ENCLOSUREMODEL}` | Modelo del recinto |
| `.6` | enclosureName | `{#ENCLOSURENAME}` | Nombre del recinto |
| `.7` | enclosureTemperature | — | Temperatura (ítem prototipo) |

### mSATAdiskTable (`.55062.2.10.32.1`) — NUEVO en QTS hero

| Columna | OID | Macro LLD | Descripción |
|---------|-----|-----------|-------------|
| `.1` | mSATAdiskIndex | `{#MSATAINDEX}` | Índice de la tabla |
| `.5` | mSATAdiskSMARTStatus | — | Estado SMART (ítem prototipo, valuemap) |
| `.6` | mSATAdiskTemperature | — | Temperatura (ítem prototipo) |
| `.8` | mSATAdiskModel | `{#MSATADISKMODEL}` | Modelo del disco mSATA |
| `.9` | mSATAdiskCapacity | — | Capacidad en MB (ítem prototipo) |

---

## 4. Value Maps — comparativa

### Value maps presentes en ambos templates

| Value Map | Template base | Este template | Diferencias |
|-----------|---------------|---------------|-------------|
| Storage Pool Status | 10 estados | 14 estados | Se añaden: SED_LOCKED, SED_LOCKING, SED_UNLOCKING, Pruning, Tuning |
| LUN Status | Sí | Sí | Sin cambios |
| SMART Status | Sí (como "QNAP Disk State") | Sí | Renombrado, mismo contenido |
| Firmware Update | Sí | Sí | Sin cambios |

### Value maps nuevos en este template

| Value Map | Descripción |
|-----------|-------------|
| QNAP Power Status | -1=Failed, 0=OK |
| QNAP Target Status | -1=Offline, 0=Ready, 1=Connected |
| QNAP Cache Type | 0=Read-only, 1=Read-write, 2=Write-only |
| QNAP WORM Status | 0=No, 1=Enterprise, 2=Compliance |
| QNAP mSATA SMART | -1=Error, 0=Good, 1=Warning, 2=Abnormal |
| QNAP JBOD Disk Status | -9=RW Error, -6=Invalid, -5=No disk, -4=Unknown, 0=Ready |

---

## 5. Funcionalidades NO disponibles en NAS.mib

Los siguientes OIDs existían en el template base usando la base legacy `24681`, pero **no tienen
equivalente documentado en la MIB QTS hero (`55062.2`)**:

### IOPS y latencia de disco

| OID legacy (24681) | Función | Estado |
|--------------------|---------|--------|
| `24681.1.4.1.11.5.6.2.1.3.1` | Disk: Overall IOPS | ❌ NO en NAS.mib |
| `24681.1.4.1.11.5.6.2.1.4.1` | Disk: Overall latency | ❌ NO en NAS.mib |

**Cómo investigar:** Si el firmware QTS hero tiene una tabla de rendimiento de disco, podría estar
en el sub-árbol `.55062.2.10` (storage). Verificar con:
```bash
snmpwalk -v2c -c <community> <IP> 1.3.6.1.4.1.55062.2.10 | grep -i iops
snmpwalk -v2c -c <community> <IP> 1.3.6.1.4.1.55062.2.10 | grep -i latency
```

### CPU por núcleo

| OID legacy (24681) | Función | Estado |
|--------------------|---------|--------|
| `24681.1.4.1.1.1.1.4.3.1.1` | CPU core discovery (LLD) | ❌ NO en NAS.mib |

**Disponible:** Solo `55062.2.12.12.0` (CPU total agregado).

**Cómo investigar:** Buscar si hay tabla de cores:
```bash
snmpwalk -v2c -c <community> <IP> 1.3.6.1.4.1.55062.2.12
```

### Interfaces de red propietarias

| OID legacy (24681) | Función | Estado |
|--------------------|---------|--------|
| `24681.1.2.9.1.x` | Network IF table propietaria | ❌ NO en NAS.mib |

**Solución implementada:** Se usa IF-MIB estándar (`1.3.6.1.2.1.2.2` y `1.3.6.1.2.1.31.1.1.1`),
que proporciona las mismas métricas (tráfico in/out, estado operativo) y es universalmente soportada.

---

## 6. Notas de implementación

### Preprocessing MULTIPLIER × 1048576

Se aplica a todos los OIDs de capacidad (discos, RAIDs, pools, volúmenes, LUNs, mSATA) bajo la
asunción de que la MIB devuelve valores en **MB**. El multiplicador convierte a bytes para que
Zabbix muestre las unidades correctamente con sus prefijos (GB, TB, etc.).

Si los valores de capacidad se muestran incorrectos tras la instalación, verificar la unidad real
del OID correspondiente con snmpwalk y ajustar el multiplicador:
- Si el OID devuelve **KB** → multiplicador `1024`
- Si el OID devuelve **GB** → multiplicador `1073741824`
- Si el OID devuelve **bytes** → eliminar el multiplicador

### Preprocessing JAVASCRIPT para estado RAID

El OID `raidStatus` devuelve un integer que se traduce a string mediante un bloque JavaScript:

```javascript
var states = {
    "-1": "Single",
    "0":  "Ready",
    "1":  "Initializing",
    "2":  "Degraded",
    "3":  "Dead",
    "4":  "Rebuilding",
    "5":  "Synchronizing",
    "6":  "Parity checking"
};
```

Los triggers de RAID usan la función `find()` sobre el texto resultante (no sobre el integer).

### Ítems calculados de % libre

Los ítems de porcentaje de espacio libre (pools, volúmenes) son de tipo CALCULATED:
```
last(/QNAP QTS hero by SNMP/storagepool.free[{#SNMPINDEX}]) /
last(/QNAP QTS hero by SNMP/storagepool.capacity[{#SNMPINDEX}]) * 100
```
Ambos ítems dependientes deben tener datos recientes para que el cálculo funcione.

### IF-MIB: uso de contadores de 64 bits

Para las interfaces de red se usan `ifHCInOctets` / `ifHCOutOctets` (contadores 64-bit, RFC 2863)
en lugar de `ifInOctets` / `ifOutOctets` (32-bit), lo que evita desbordamientos en interfaces
de 1G/10G con tráfico elevado.

### storagepoolStatus: valores para QuHERO vs QuTS

Según el análisis de la MIB, el enum `storagepoolStatus` tiene rangos de valores distintos según
el firmware. Los 14 valores mapeados en el template cubren ambos casos documentados en NAS.mib.

---

## 7. Resumen estadístico del template

| Componente | Cantidad |
|------------|----------|
| Value maps | 10 |
| Macros configurables | 13 |
| Ítems estáticos | 26 (incluye 1 calculado) |
| LLD Rules | 10 |
| Ítems prototipo (total) | 26 |
| Trigger prototipos (total) | 18 |
| Triggers escalares | 10 |
| **Total triggers** | **28** |
