# Handoff PMIE 2026 — Punto de partida para continuar el proyecto

**Fecha de corte:** 22 de mayo de 2026
**Branch activo:** `claude/continue-project-tool-VQTZE`
**Repo:** `chlopezr-personalsp/chlopezr-personalsp.github.io`
**Último commit:** `edc898a` — Doc: actualizaciones del acta del 24 de mayo de 2026

---

## 1. Contexto del proyecto

Plan Maestro de Infraestructura Educativa (PMIE) 2026 para una Secretaría de Educación. El producto está compuesto por:

1. **Herramienta dinámica HTML** (`PMIE2026_Herramienta_Consolidada.html`, ~5.500 líneas, monolítica). Recibe Excel de matrícula y de brecha funcional/estructural, ejecuta un árbol de decisión de 11 nodos (N1–N5) y asigna sedes a 8 programas (P1–P8). Genera dashboard + Excel descargable.
2. **Documento técnico** (`PMIE_2026_Plan_Maestro_Tecnico_Final_v2.docx`) — Plan Sectorial formal con anexos A–G.
3. **Presentación ejecutiva** (`PMIE_2026_Presentacion_Ejecutiva.pptx`) — 22 diapositivas widescreen.

---

## 2. Arquitectura de la herramienta (HTML monolítico)

| Función clave | Línea aprox. | Qué hace |
|---|---|---|
| `_calcularBrechaFuncional` | 851 | Calcula índice NTC 4595 (70% disponibilidad + 30% estado, con factor de penalización por mal estado) |
| `_calcularCoberturaNOOficial` | 1067 | Distribuye matrícula NO OFICIAL en sedes virtuales de 1.400 estudiantes por comuna |
| `runTree` | 1149 | Árbol de decisión N1→N5; respeta bloqueo de zonas verdes POT |
| `renderKPIs` | 1637 | Dashboard con 3 escenarios temporales (2024/2026/2039) |
| `_buildExcel` | 2065 | Genera Excel con hojas P1–P8 + RESUMEN, columnas Traslado MEP y Dotación |
| `_procesarBaseConsolidadaBF` | 5424 | Detecta dinámicamente columna ZONA_VERDE en headers |
| `_recalcularIPC` | (cerca de COSTOS_2026) | Recalcula `IPC_H3` dinámico = `(1 + inflación)^(año_obj − 2026)` |

**Parámetros UI configurables:**
- `paramQCorte` — quintil de corte (default **Q3** desde acta 24/05/2026)
- `paramInflacionAnual` — inflación sector construcción (default **4,5%**)
- `paramAnoProyeccion` — año horizonte (default 2039)

**Constantes embebidas relevantes:**
- `ESTUDIANTES_POR_SEDE_NUEVA = 1400`
- `FACTOR_DOTACION = 0.10`
- `MEPCostXm2_mes = 25000` COP — costo mensual de Módulos Educativos Prefabricados
- `meses_obra = 18` (P8) / `12` (P4 vía N3b-SÍ)

---

## 3. Cambios aplicados por el acta del 24 de mayo de 2026

Acta de referencia: `260524_ACTA_4143.040.9.33.43_MESA_TRABAJO_PMIE.docx`. Implementados en commits `4f3295d` (herramienta) y `edc898a` (documento).

### Críticos
- **A1** Q-Corte default Q2 → Q3
- **A2** Dashboard muestra NO OFICIAL en 3 escenarios (2024: 19.785 / 2026: 42.771 / 2039: proyección lineal ~20.000)
- **A3** Nota técnica en Anexo G.16 sobre discrepancia $6,5M vs $7,29M por m² (referencia simplificada vs cálculo riguroso con FD + IPC + overhead 14%)
- **A4** Distribución de cobertura NO OFICIAL en sedes virtuales de 1.400 estudiantes por comuna

### Metodológicos
- **B1** Índice NTC 4595 ahora ponderado 70% disponibilidad + 30% estado (antes 100% estado)
- **B2** Penalización 15–40% del índice cuando estado promedio < 2,5/4

### Nuevas funcionalidades
- **C1** Columna ZONA_VERDE en BASE CONSOLIDADA BRECHA FUNCIONAL bloquea P1/P4 y fuerza P5
- **C2** Costo de Traslado MEP en programas con demolición (P8 / P4-N3b-SÍ)
- **C3** Dotación 10% sobre costo de construcción nueva en P1/P4-N3b/P8
- **C4** Inflación anual configurable reemplaza IPC_H3 hardcoded

### Documentación
- **D1** Dashboard reorganizado: bloque BRECHAS (BF/BE/BC) + bloque PLAN (sedes/P6/cobertura/TOTAL)
- **D2** Lenguaje institucional ("metodología de clasificación técnica" complementa "runTree")
- **D3** Anexo G.10–G.17 con tabla de pendientes operativos

---

## 4. Pendientes operativos del acta (sin implementación de código)

| Compromiso | Responsable | Fecha límite |
|---|---|---|
| Completar levantamiento de 4 sedes (General Santander) | Equipo de levantamiento | 15/05/2026 |
| Consolidar listado definitivo de zonas verdes POT | Carlos López + Diego Dorado | 31/05/2026 |
| Ajustar lenguaje institucional del plan | Carlos López | Junio 2026 |
| Incluir información de dotación de equipamiento | Equipo SED | Fase siguiente |
| Validar costos 2026 e inflación sector construcción | Carlos López + Juan Carlos | Próxima reunión |

---

## 5. Validaciones esperadas tras los cambios

1. **Carga sin errores:** Excel BF con columna ZONA_VERDE → consola imprime `[BF] N sedes en zona verde`.
2. **Q-Corte=Q3 default:** ~177 sedes en Q1-Q3 (vs ~117 con Q1-Q2).
3. **Cobertura NO OFICIAL 2026:** ~31 sedes virtuales nuevas para 42.771 estudiantes.
4. **Plan total:** ~$5–5,5 B (subida por mayor cobertura + dotación + MEP) vs BC ~$2,87 B.
5. **Sedes en zona verde:** nunca aparecen en P4 ni en P1 oficial; `prog_array` queda como `[P5, P3]` o similar.
6. **Smoke test JS:**
   ```bash
   python3 -c "import re; c=open('PMIE2026_Herramienta_Consolidada.html').read(); open('/tmp/js','w').write(''.join(re.findall(r'<script(?![^>]*\\bsrc=)[^>]*>(.*?)</script>', c, re.DOTALL)))"
   node --check /tmp/js && echo OK
   ```

---

## 6. Archivos clave en el repo

| Archivo | Estado |
|---|---|
| `PMIE2026_Herramienta_Consolidada.html` | Última versión con A1–A4 + B1–B2 + C1–C4 |
| `PMIE_2026_Plan_Maestro_Tecnico_Final_v2.docx` | Versión vigente — incluye Anexo G.10–G.17 |
| `PMIE_2026_Plan_Maestro_Tecnico_Final.docx` | Versión anterior (G.1–G.9) — conservar como referencia |
| `PMIE_2026_Presentacion_Ejecutiva.pptx` | **Pendiente** sincronizar slides 11 y 13 con cifras nuevas |
| `index.html` | Landing page del GitHub Pages |

---

## 7. Próximos pasos sugeridos

1. **Actualizar PPTX** — regenerar slides 11 (Brechas) y 13 (sedes/programa) con valores post-acta:
   - BC ≈ $2,87 B (sin cambio)
   - Plan total ≈ $5,0–5,5 B (antes $4,27 B)
   - ~31 sedes virtuales NO OFICIAL (antes 14)
2. **Validar con datos reales** — correr la herramienta con la versión final de `BASE CONSOLIDADA BRECHA FUNCIONAL` que incluya la columna ZONA_VERDE definitiva.
3. **Calibrar inflación** — confirmar con Hacienda el valor 2026 (default 4,5% es estimado).
4. **Probar bloqueo zona verde** — verificar con sedes reales que el árbol fuerza P5 + P3/P7 y nunca asigna P1/P4.
5. **Audit log en la UI** — opcional: registrar en el dashboard cuántas sedes fueron redirigidas por el bloqueo zona verde (trazabilidad).

---

## 8. Comandos útiles para retomar

```bash
# Estado del repo
cd /home/user/chlopezr-personalsp.github.io
git status
git log --oneline -10

# Validar sintaxis JS de la herramienta
python3 -c "import re; c=open('PMIE2026_Herramienta_Consolidada.html').read(); open('/tmp/js','w').write(''.join(re.findall(r'<script(?![^>]*\\bsrc=)[^>]*>(.*?)</script>', c, re.DOTALL)))" && node --check /tmp/js

# Servir localmente para probar
python3 -m http.server 8000  # abrir http://localhost:8000/PMIE2026_Herramienta_Consolidada.html
```

---

## 9. Convenciones del proyecto

- Cifras en pesos colombianos con separador de miles (1.234.567) y unidades explícitas (MM COP, m², estudiantes).
- Siglas BF / BE / BC se expanden en su primer uso por sección (Brecha Funcional / Brecha Estructural / Brecha Consolidada).
- Programas P1–P8 mantienen su numeración oficial; los nodos N1–N5 también.
- Todos los commits van al branch `claude/continue-project-tool-VQTZE`.
