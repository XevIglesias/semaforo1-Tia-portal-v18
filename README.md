# Semáforo por etapas (GRAFCET → LAD) en TIA Portal V18 — S7‑1500

Proyecto didáctico de PLC realizado por **Xavier Iglesias** en **Siemens TIA Portal V18 (Student)**.  
Implementa un **semáforo de 3 luces** usando una **máquina de estados** de 4 etapas (**E0–E3**) diseñada con **GRAFCET** y programada en **Ladder (LAD)**.

---

## 1) Objetivo
- Controlar un semáforo con **3 salidas**: verde, naranja y rojo.
- Secuenciar el comportamiento mediante **4 etapas**:
  - **E0**: reposo
  - **E1**: verde
  - **E2**: naranja
  - **E3**: rojo
- Permitir control por operador:
  - **Marcha** inicia el ciclo
  - **Paro** fuerza vuelta a reposo (E0) desde cualquier etapa

---

## 2) Entorno
- **PLC**: Siemens **S7‑1500** — **CPU 1512C‑1 PN**
- **Software**: **TIA Portal V18 (Student)**
- **Lenguaje**: **LAD (Ladder)**

---

## 3) Temporización (final)
- **Verde:** 45 s  
- **Naranja:** 3 s  
- **Rojo:** 20 s  

> Los temporizadores se implementan en el PLC (son TP, temporizador de impulsos).

---

## 4) Secuencia (GRAFCET)
El control se diseñó primero con un **GRAFCET** y luego se implementó en LAD.

**Etapas**
- **E0 (Reposo):** espera la orden de Marcha.
- **E1 (Verde):** se activa `L_VERDE` durante 45 s.
- **E2 (Naranja):** se activa `L_NARANJA` durante 3 s.
- **E3 (Rojo):** se activa `LROJO` durante 20 s.
- Al finalizar la etapa **E3**, el sistema vuelve a **E0** (reposo).

Diagrama: ver `docs/01_grafcet.pdf`.

---

## 5) Tabla de tags (PLC Tags)

> Tipos: todos **BOOL** excepto `System_Byte` (BYTE).

### Entradas (Inputs)
| Tag | Dirección | Tipo | Descripción |
|---|---:|---|---|
| `Marcha` | `I0.0` | BOOL | Orden de marcha |
| `Paro` | `I0.1` | BOOL | Orden de paro |

### Etapas (Memorias)
| Tag | Dirección | Tipo | Descripción |
|---|---:|---|---|
| `E0` | `M0.0` | BOOL | Etapa 0: Reposo |
| `E1` | `M0.1` | BOOL | Etapa 1: Verde |
| `E2` | `M0.2` | BOOL | Etapa 2: Naranja |
| `E3` | `M0.3` | BOOL | Etapa 3: Rojo |

### Salidas (Outputs)
| Tag | Dirección | Tipo | Descripción |
|---|---:|---|---|
| `L_VERDE` | `Q0.0` | BOOL | Luz verde |
| `L_NARANJA` | `Q0.1` | BOOL | Luz naranja |
| `LROJO` | `Q0.2` | BOOL | Luz roja |

### System Memory (bits)
| Tag | Dirección | Tipo | Descripción |
|---|---:|---|---|
| `System_Byte` | `MB100` | BYTE | System byte (System Memory) |
| `First_Scan` | `M100.0` | BOOL | Primer ciclo (First Scan) |
| `Diag_Status_Update` | `M100.1` | BOOL | Diagnóstico / status update |
| `Always_True` | `M100.2` | BOOL | Siempre TRUE |
| `Always_False` | `M100.3` | BOOL | Siempre FALSE |

Captura de tags: ver `docs/02_tags.pdf`.

---

## 6) Estructura de la programación (OB1)
La lógica se implementa en el **OB1** en LAD y se organiza conceptualmente en:

1. **Red(es) de etapas/transiciones**
   - Uso de `First_Scan` para asegurar que el sistema arranca en **E0**.
   - Transiciones condicionadas por:
     - Marcha/Paro
     - Finalización de temporizadores
     - Estado de la etapa actual/anterior

2. **Red(es) de acciones (salidas)**
   - `E1` activa `L_VERDE`
   - `E2` activa `L_NARANJA`
   - `E3` activa `LROJO`

## Prueba rápida
1. Activar `Marcha (I0.0)`.
2. Ver secuencia:
   - Verde `Q0.0` durante 45 s
   - Naranja `Q0.1` durante 3 s
   - Rojo `Q0.2` durante 20 s
3. `Paro (I0.1)` fuerza retorno a `E0 (M0.0)`.
   
## 7) Simulación y plan de pruebas (Test Plan)

### Prueba 1 — Arranque en frío (First Scan)
- Acción: iniciar simulación/CPU.
- Esperado: sistema en **E0** y estado seguro en salidas (según diseño).

### Prueba 2 — Inicio de ciclo (Marcha)
- Acción: activar `Marcha (I0.0)`.
- Esperado: entra en **E1** y se enciende `L_VERDE (Q0.0)`.

###
