# Contratos Calamar — Contexto del Proyecto

Análisis de inversión + dashboard para due diligence de contratos de obra civil pública en Calamar, Colombia. Stiven (cuñado de Juan) evalúa entrar como socio financiero del consorcio que ejecuta Emily.

---

## STACK

- **Frontend:** HTML/CSS/JS vanilla en un solo archivo (`index.html`)
- **Charts:** Chart.js 4.4 vía CDN
- **DB:** Neon Postgres serverless (proyecto `proyecto-calamar`, host `ep-divine-fire-acxhppmp.sa-east-1.aws.neon.tech`) consultado directo desde el browser con `@neondatabase/serverless@1` (esm.sh)
- **Wrapper SQL:** `sb` que imita la API de Supabase para no reescribir queries legacy (ver `index.html` `// ── NEON SERVERLESS CLIENT`)
- **Storage de archivos:** base64 en columnas `archivo_data` (NO usa Supabase Storage)
- **Hosting:** GitHub Pages — repo `juandlarrartec-beep/contratos-calamar`
- **URL pública:** https://juandlarrartec-beep.github.io/contratos-calamar/

---

## TABLAS NEON

```sql
respuestas (cat_idx, item_idx, texto, checked, archivo_*, updated_at)
pagos (id, fecha, concepto, categoria, quien, monto, metodo_pago, proveedor, notas, archivo_*)
conciliaciones (id, fecha, concepto, categoria, proveedor, nro_factura, metodo_pago, monto, notas, detalles JSONB, archivo_*)
```

El bootstrap del HTML aplica `ALTER TABLE ADD COLUMN IF NOT EXISTS` para columnas que pueden faltar — es idempotente y seguro.

---

## ESTRUCTURA DEL `index.html`

### Tabs (orden actual definido por Juan 2026-05-12)
1. **Capital** — pagos invertidos por Stiven (tabla `pagos`, dinámica)
2. **Conciliación** — gastos que Emily reporta haber hecho (tabla `conciliaciones`)
3. **Resumen** — vista ejecutiva con números clave + charts + cronograma
4. **Finanzas** — distribución utilidad + protecciones
5. **Contratos** — los 3 contratos del consorcio + contrato pinned PDF
6. **Riesgo** — riesgos identificados con mitigaciones
7. **Preguntas** (oculto-ish, opacity .7) — due diligence original 30 preguntas

### Features dinámicas
- Cada fila de Capital y Conciliación tiene **botón ✎ Edit** (popula form + scroll)
- **Re-upload de archivo:** si en edit no se adjunta nuevo, queda el actual
- **Categorías y métodos dinámicos:** opción `+ Agregar nueva...` en los 4 selects, persiste en localStorage por scope (`calamar-cats-pago`, `calamar-cats-conc`, `calamar-metodos`)
- **parseMonto():** acepta `12.000.000`, `12,000,000`, `$12 000 000` (parseInt simple truncaba al primer separador)

---

## FRAMEWORK FINANCIERO (corregido 2026-05-12 vía audio Stiven)

**Importante:** los porcentajes de participación NO se aplican sobre el valor del contrato sino sobre la **utilidad real**.

```
Utilidad real = Valor contrato − Gastos totales

Gastos estimados (audio Stiven 12/05):
  Construcción mano de obra (6 colegios × $40M)  $240M
  Ornamentación ($20M × 6)                       $120M
  Materiales (~18% obra)                         $200M
  Gastos administrativos ($30M × 6 meses)        $180M
                                                ─────
  TOTAL                                          $740M

Utilidad: $1.084M − $740M = $344M (Stiven dijo $364M con su suma a $720M)

Distribución (estructura 70/30):
  70% socios silentes:  ~$247.8M
  30% pool Emily+Stiven: ~$106.2M
    Stiven 15% sobre utilidad: ~$53.1M bruto
```

### Costo financiero del préstamo
$50M al 5% mensual × 6 meses = $15M. **Se reparte 50/50 entre Emily y Stiven** ($7.5M c/u). Decisión de Juan 2026-05-12.

### Retorno final estimado por socio
- **Emily:** +$80M (etapa 1) + $53.1M (utilidad) − $7.5M (préstamo) = **~$125.6M**
- **Stiven:** +$100M (capital) + $53.1M (utilidad) − $7.5M (préstamo) = **~$145.6M** · ROI sobre $100M ~46% en 6 meses

### Margen oculto detectado
Emily infla facturas del maestro de obra: cobra $120M, factura $150–160M. Diferencia $30–40M × 2 bloques = **$60–80M desviados** antes de calcular utilidad. Si el % de Stiven aplica sobre utilidad contable, pierde ese margen. El contrato escrito debe blindar utilidad bruta con derecho de auditoría.

---

## ACTORES

| Nombre | Rol |
|---|---|
| **Stiven** | Cuñado de Juan. Potencial inversor financiero. Capital prestado $50M al 5% mensual desde 06/03/2026. |
| **Emily** | Contratista principal del consorcio. Maneja ejecución técnica y nómina. |
| **Alejandra** | Co-socia de Emily. Arma reportes de nómina. |
| **3 socios silentes** | 70% del consorcio. NO identificados aún — riesgo abierto. |

---

## ESTADO ACTUAL (al 2026-05-12)

- **Contrato firmado:** 5 mar 2026 (#1380-2097-2025, valor $1.084M)
- **Cronograma:** pasos 1-3 completos (contrato firmado, diagnóstico, presupuestación aprobada). **Paso 4 activo: esperando anticipo del Estado (20% = $216.6M).**
- **Capital invertido por Stiven:** ~$96.26M (21 pagos en Neon)
- **Gastos conciliados por Emily:** ~$57.82M (4 conciliaciones) → 60% de conciliación
- **Pendiente conciliar:** ~$38.44M

---

## ARCHIVOS CLAVE

- `index.html` — dashboard completo (1700+ líneas)
- `proyecto.json` — datos iniciales del due diligence
- `analisis/00_RESUMEN_EJECUTIVO.txt` — resumen ejecutivo actualizado 12/05
- `analisis/02_Valor_Flujo_Caja_Retorno/analisis.txt` — análisis financiero por pregunta
- `analisis/08_Audio_Stiven_12_05_2026/` — transcripción del audio + análisis de costos realistas (3 escenarios)
- `analisis/01_Legalidad_y_Estructura/CONTRATO_DE_OBRA_1380-2097-2025.pdf` — contrato oficial pineado en el dashboard

---

## CONVENCIONES

- **Idioma:** español en todo (UI, comentarios, commits)
- **NO Supabase** — migrado a Neon en abril 2026. El wrapper `sb` es nomenclatura legacy
- **Archivos en DB** como base64 (no usa storage externo). Validación: 10MB max, JPG/PNG/WEBP/PDF
- **Commits:** prefijo `feat:` `fix:` `chore:` en español, con `Co-Authored-By: Claude Opus 4.7 (1M context)`
- **Deploy:** `git push origin main` → GitHub Pages rebuild automático ~1min

---

## ESTILO DE TRABAJO CON JUAN EN ESTE PROYECTO

- Juan transcribe audios de Stiven con ElevenLabs Scribe y los integra al análisis
- Cuando Juan dice "actualizá los números" usar el framework correcto (% sobre utilidad, no sobre contrato)
- Cualquier referencia a inversión de Emily debe aclarar "(costos estimados — falta verificación oficial)"
- Los gastos pre-inicio fijos NO van más en el HTML — todo el tracking de capital es dinámico desde Neon
- Cuando se agreguen features al HTML, deploy = commit + push automático (Juan confirmó)
