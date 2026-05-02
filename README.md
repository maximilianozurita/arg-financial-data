# arg-financial-data

Dataset público de indicadores económicos y financieros de Argentina.
Actualizado automáticamente desde [`arg-financial-local`](https://github.com/maximilianozurita/arg-financial-local).

---

## Series disponibles

| Categoría | Serie | Fuente | Frecuencia | Desde |
|-----------|-------|--------|-----------|-------|
| acciones | Índice Merval | Yahoo Finance | Diaria | 2000 |
| cambiario | Tipo de Cambio Minorista BCRA | BCRA | Diaria | 2003 |
| cambiario | Dólar Blue compra / venta | Bluelytics | Diaria | 2011 |
| cambiario | Dólar Oficial compra / venta | Bluelytics | Diaria | 2011 |
| cambiario | Dólar MEP compra / venta | ArgentinaDatos | Diaria | 2018 |
| cambiario | Dólar CCL compra / venta | ArgentinaDatos | Diaria | 2013 |
| monetario | Reservas Internacionales BCRA | BCRA | Diaria | 1994 |
| monetario | Base Monetaria | BCRA | Diaria | 1994 |
| monetario | BADLAR Bancos Privados | BCRA | Diaria | 2001 |
| monetario | Tasa de Política Monetaria | BCRA | Diaria | 2017 |
| monetario | Tasa Pases Pasivos BCRA 1 día | BCRA | Diaria | 2014 |
| financiero | Riesgo País (EMBI+) | ArgentinaDatos | Diaria | 1999 |
| precios | IPC - Nivel General | INDEC | Mensual | 2017 |
| precios | Inflación Esperada 12m (REM) | BCRA | Diaria | 2004 |
| actividad | EMAE - Actividad Económica | INDEC | Mensual | 2004 |
| actividad | IPI Manufacturero | INDEC | Mensual | 2016 |
| laboral | Índice de Salarios | INDEC | Mensual | 2016 |
| fiscal | Recaudación Tributaria | Ministerio de Economía | Mensual | 1997 |
| fiscal | Gasto Corriente del Estado | Ministerio de Economía | Trimestral | — |
| fiscal | Gasto Público Nacional | Ministerio de Economía | Anual | — |
| fiscal | Gasto Público Consolidado | Ministerio de Economía | Anual | — |
| social | Línea de Pobreza | INDEC | Mensual | 2016 |
| social | Línea de Indigencia | INDEC | Mensual | 2016 |

El catálogo completo y actualizado está en [`metadata.json`](./metadata.json).

## Estructura

```
arg-financial-data/
├── data/
│   ├── acciones/
│   ├── cambiario/
│   │   ├── dolar_blue_venta.csv
│   │   ├── dolar_blue_venta.parquet
│   │   └── ...
│   ├── monetario/
│   ├── precios/
│   ├── actividad/
│   ├── laboral/
│   ├── financiero/
│   ├── fiscal/
│   └── social/
├── all_series.csv       ← todas las series unificadas
├── all_series.parquet   ← todas las series unificadas
├── metadata.json        ← catálogo de series con metadatos y slugs
└── latest.json          ← último valor de cada serie
```

### Formato CSV (series individuales)

```
fecha,valor
2024-01-02,815.00
2024-01-03,820.00
```

### Formato all_series

```
fecha,nombre,fuente,categoria,unidad,frecuencia,valor
2024-01-02,Dólar Blue (Venta),Bluelytics,cambiario,ARS/USD,diaria,820.0
```

### Formato Parquet

Mismas columnas que CSV. Recomendado para series largas — hasta 10x más compacto.

## Consumo

### Python / pandas

```python
import pandas as pd

BASE = "https://raw.githubusercontent.com/maximilianozurita/arg-financial-data/main"

# Serie individual
df = pd.read_csv(f"{BASE}/data/cambiario/dolar_blue_venta.csv", parse_dates=["fecha"])

# Todas las series
df = pd.read_parquet(f"{BASE}/all_series.parquet")

# Filtrar
blue = df[df["nombre"] == "Dólar Blue (Venta)"]
```

### DuckDB

```sql
SELECT * FROM read_parquet(
  'https://raw.githubusercontent.com/maximilianozurita/arg-financial-data/main/data/cambiario/dolar_blue_venta.parquet'
)
WHERE fecha >= '2023-01-01'
ORDER BY fecha DESC;
```

### JavaScript / TypeScript

```js
const BASE = "https://raw.githubusercontent.com/maximilianozurita/arg-financial-data/main"

const metadata = await fetch(`${BASE}/metadata.json`).then(r => r.json())
const latest   = await fetch(`${BASE}/latest.json`).then(r => r.json())
const csv      = await fetch(`${BASE}/data/cambiario/dolar_blue_venta.csv`).then(r => r.text())
```

## Actualización

| Tipo | Frecuencia |
|------|-----------|
| Fuentes diarias (BCRA, Bluelytics, ArgentinaDatos, Merval) | Lunes a viernes ~8:00 hs (UTC-3) |
| Fuentes mensuales (INDEC, MECON) | Día 6 de cada mes ~8:00 hs (UTC-3) |

## Fuentes

Todas públicas, sin autenticación.

| Fuente | API |
|--------|-----|
| [BCRA](https://www.bcra.gob.ar) | `api.bcra.gob.ar/estadisticas/v4.0` |
| [Bluelytics](https://bluelytics.com.ar) | `api.bluelytics.com.ar/v2` |
| [ArgentinaDatos](https://argentinadatos.com) | `api.argentinadatos.com/v1` |
| [INDEC](https://www.indec.gob.ar) | `apis.datos.gob.ar/series/api` |
| [Ministerio de Economía](https://www.economia.gob.ar) | `apis.datos.gob.ar/series/api` |
| [Yahoo Finance](https://finance.yahoo.com) | `yfinance` (Python) |
