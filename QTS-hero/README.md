# QNAP QTS hero / QuTS hero — Template Zabbix 7.2 SNMP

Template SNMPv2c para monitorización completa de NAS QNAP con firmware **QuTS hero** (h5.x+) en **Zabbix 7.2**.

> **Importante:** Este template NO es compatible con firmware QTS clásico (4.x/5.x).
> QTS clásico usa la base OID `1.3.6.1.4.1.24681` o `1.3.6.1.4.1.55062.1.x`.
> QuTS hero usa `1.3.6.1.4.1.55062.2.x` — bases incompatibles.

---

## Requisitos

| Componente | Versión mínima |
|------------|---------------|
| Zabbix Server | 7.2 |
| Firmware QNAP | QuTS hero h5.x+ |
| SNMP | v2c habilitado en el NAS |
| Protocolo | SNMPv2c (este template no usa SNMPv3) |

---

## Funcionalidades

### Ítems estáticos (26 ítems)

| Categoría | Métricas |
|-----------|----------|
| **Sistema** | Uptime, modelo, hostname, número de serie, firmware, temperatura CPU/sistema, uso CPU, memoria total/usada/% |
| **Alimentación** | Estado de alimentación, código estado UPS |
| **UPS externo** | Estado textual, % batería, % carga |
| **Caché mSATA** | % disponible, tasa hit lectura/escritura, tipo, modo, tipo RAID |
| **Contadores** | Número de discos, RAIDs y pools |

### Low Level Discovery (10 reglas)

| Regla LLD | OID base | Ítems prototipo | Triggers prototipo |
|-----------|----------|-----------------|-------------------|
| Disk Discovery | `.55062.2.10.2` | SMART status, temperatura, capacidad | SMART error (DISASTER), SMART warning (HIGH), temp HIGH, temp WARN |
| Fan Discovery | `.55062.2.12.9` | Velocidad (RPM) | Ventilador parado (HIGH) |
| RAID Discovery | `.55062.2.10.5` | Estado, capacidad, nivel | Degraded/Dead (DISASTER), Rebuilding/Sync (WARNING) |
| Storage Pool Discovery | `.55062.2.10.7` | Estado, capacidad, espacio libre, % libre | Not Ready (HIGH), espacio crítico (AVERAGE), espacio bajo (WARNING) |
| Volume Discovery | `.55062.2.10.9` | Estado, capacidad, espacio libre, % libre | Not Ready (DISASTER), espacio crítico (AVERAGE), espacio bajo (WARNING) |
| iSCSI LUN Discovery | `.55062.2.10.20` | Estado, capacidad, mapped status | Not Enabled/Ready (HIGH) |
| iSCSI Target Discovery | `.55062.2.10.22` | Estado | Offline (HIGH) |
| Network IF Discovery | IF-MIB estándar | Bits in/out (bps), estado operativo | Link down (AVERAGE) |
| Enclosure Discovery | `.55062.2.10.34` | Temperatura | Temperatura alta (WARNING) |
| mSATA Cache Disk Discovery | `.55062.2.10.32` | SMART status, temperatura, capacidad | SMART error (HIGH) |

### Triggers escalares (10 triggers)

| Trigger | Severidad |
|---------|-----------|
| SNMP: No response from host | DISASTER |
| System: Device restarted (uptime < 10m) | WARNING |
| System: Firmware update available | INFO |
| System: Power failure detected | HIGH |
| CPU: Temperature too high (>85°C / 15m) | HIGH |
| CPU: Temperature warning (>75°C / 15m) | WARNING |
| System: Temperature too high | HIGH |
| System: Temperature warning | WARNING |
| UPS: Battery charge low (<30%) | WARNING |
| Memory: High utilization (>90% / 15m) | WARNING |

### Value Maps (10 mapeos)

- QNAP Power Status
- QNAP Storage Pool Status (14 estados ZFS)
- QNAP LUN Status
- QNAP Target Status
- QNAP SMART Status
- QNAP Cache Type
- QNAP Firmware Update
- QNAP WORM Status
- QNAP mSATA SMART
- QNAP JBOD Disk Status

---

## Instalación

### 1. Habilitar SNMP en el NAS

En QTS hero: **Panel de control → Red y conmutador virtual → SNMP**

- Habilitar servicio SNMP
- Versión: SNMPv2c (o SNMPv3 para mayor seguridad, aunque este template usa v2c)
- Community string: configurar la deseada (por defecto `public`)
- Puerto: 161 (UDP, estándar)

### 2. Descargar la MIB (opcional, para resolución de nombres)

Descargar el archivo MIB de QNAP e instalarlo en el servidor Zabbix para que los OIDs se resuelvan a nombres legibles:

```bash
# Copiar NAS.mib al directorio de MIBs de Zabbix
cp NAS.mib /usr/share/snmp/mibs/
# o
cp NAS.mib /usr/share/zabbix/mibs/
```

### 3. Importar el template en Zabbix

**Zabbix Web UI → Configuración → Templates → Importar**

- Seleccionar el archivo `template_qnap_qts_hero_zabbix72.yaml`
- Marcar las opciones de creación/actualización según necesidad
- Confirmar la importación

### 4. Crear el host en Zabbix

1. **Configuración → Hosts → Crear host**
2. Configurar la interfaz SNMP:
   - Tipo: SNMP
   - Dirección IP: IP del NAS QNAP
   - Puerto: 161
   - Versión SNMP: SNMPv2c
   - Community: `{$SNMP_COMMUNITY}` (o el valor directo si no usas macro)
3. Vincular el template: **Templates → QNAP QTS hero by SNMP**

### 5. Ajustar macros

En el host o a nivel global, ajustar las macros según el entorno:

| Macro | Valor por defecto | Descripción |
|-------|-------------------|-------------|
| `{$SNMP_COMMUNITY}` | `public` | Community string SNMP |
| `{$SNMP.TIMEOUT}` | `5m` | Tiempo sin respuesta → DISASTER |
| `{$CPU_TEMP_WARN}` | `75` | Temperatura CPU WARNING (°C) |
| `{$CPU_TEMP_HIGH}` | `85` | Temperatura CPU HIGH (°C) |
| `{$HDD_TEMP_WARN}` | `45` | Temperatura HDD WARNING (°C) |
| `{$HDD_TEMP_HIGH}` | `55` | Temperatura HDD HIGH (°C) |
| `{$POOL_FREE_WARN}` | `20` | % libre pool WARNING |
| `{$POOL_FREE_CRIT}` | `10` | % libre pool CRITICAL |
| `{$VOL_FREE_WARN}` | `20` | % libre volumen WARNING |
| `{$VOL_FREE_CRIT}` | `10` | % libre volumen CRITICAL |
| `{$UPS_BATT_WARN}` | `30` | % batería UPS WARNING |
| `{$NET_IF_MATCHES}` | `.+` | Regex inclusión interfaces |
| `{$NET_IF_NOT_MATCHES}` | `CHANGE_IF_NEEDED` | Regex exclusión interfaces |

---

## Validación (checklist snmpwalk)

Antes de importar el template, verificar la conectividad SNMP y que los OIDs responden:

```bash
# Conectividad básica — debe devolver datos del árbol QTS hero
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2

# Sistema
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.3.0   # model
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.4.0   # hostname
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.6.0   # firmware version
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.10.0  # cpu temperature
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.11.0  # system temperature
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.12.0  # cpu usage %
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.13.0  # memory total (MB)
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.16.0  # memory used (MB)
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.19.0  # power status
snmpget -v2c -c public <IP_NAS> 1.3.6.1.2.1.1.3.0            # uptime (MIB-II)

# LLD — tablas de descubrimiento
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.2    # discos
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.5    # RAID
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.7    # pools ZFS
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.9    # carpetas compartidas
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.20   # LUNs iSCSI
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.22   # targets iSCSI
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.12.9    # ventiladores
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.34   # recintos expansión
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.32   # discos mSATA
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.2.1.2.2             # interfaces IF-MIB

# UPS (solo si hay UPS conectado)
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.13.1.0   # ups status text
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.13.6.0   # ups load %
snmpget -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.13.7.0   # ups battery charge %
```

---

## Notas sobre unidades de capacidad

Los OIDs de capacidad en la MIB QTS hero devuelven valores enteros que se asumen en **MB**.
El template aplica un multiplicador de `1048576` (1 MB = 1048576 bytes) para almacenar en Zabbix las capacidades en bytes.

Si tras la instalación observas valores de capacidad incorrectos, verificar con snmpwalk la unidad real:
```bash
snmpwalk -v2c -c public <IP_NAS> 1.3.6.1.4.1.55062.2.10.2.1.9  # diskCapacity
```
Ajustar el multiplicador en los ítems de capacidad si la MIB devuelve GB o KB en lugar de MB.

---

## Limitaciones conocidas

Los siguientes OIDs **no están disponibles** en la MIB QTS hero y no pueden monitorizarse:

| Funcionalidad | Motivo |
|---------------|--------|
| IOPS global de disco | OID `24681.1.4.1.11.5.6.2.1.3` es legacy QTS, no existe en MIB 55062.2 |
| Latencia global de disco | OID `24681.1.4.1.11.5.6.2.1.4` es legacy QTS, no existe en MIB 55062.2 |
| CPU usage por núcleo | Solo CPU total disponible en MIB (`.55062.2.12.12.0`) |

Consultar `MAPPING.md` para el mapeo completo de todos los OIDs.

---

## Diferencias respecto al template base Zabbix 6.x comunitario

| Aspecto | Template base (Zabbix 6.x) | Este template (Zabbix 7.2) |
|---------|---------------------------|---------------------------|
| Base OID | `24681.x` + `55062.1.x` (mixta) | `55062.2.x` (QTS hero exclusivo) |
| Zabbix mínimo | 6.0 | 7.2 |
| Firmware compatible | QTS 4.5.2+ | QuTS hero h5.x+ |
| CPU por núcleo | Sí (LLD) | No disponible en MIB |
| IOPS/Latencia globales | Sí | No disponible en MIB |
| Recintos de expansión | No | Sí (nuevo LLD) |
| Discos mSATA | No | Sí (nuevo LLD) |
| Memoria RAM | No | Sí (total, usada, % utilización) |
| UPS detallado | Básico | Estado, batería, carga |

---

## Archivos del proyecto

```
zabbix-7.2/QTS-hero/
├── template_qnap_qts_hero_zabbix72.yaml  ← Template principal (importar en Zabbix)
├── README.md                              ← Este archivo
└── MAPPING.md                             ← Mapeo completo de OIDs (referencia técnica)
```

---

## Referencias

- [QNAP MIB oficial QuTS hero](https://www.qnap.com/en/product/catalog.php) — descargar desde la interfaz web del NAS
- [Zabbix 7.2 documentation — SNMP](https://www.zabbix.com/documentation/7.2/en/manual/config/items/itemtypes/snmp)
- [IF-MIB RFC 2863](https://tools.ietf.org/html/rfc2863)
- Template base de referencia: [albin-lindstrom/zabbix-qnap-template](https://github.com/albin-lindstrom/zabbix-qnap-template)
