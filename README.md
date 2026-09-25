# hidroestimador

Pipeline en R para descargar, procesar y extraer datos de lluvia del Global Hydro-Estimator (GHE) de NOAA, y compararlos con registros de estaciones meteorológicas en tierra.

El GHE es un producto satelital que estima la precipitación a partir de imágenes infrarrojas, con una resolución espacial de ~5 km y una resolución temporal de 15 minutos. Este repositorio contiene el flujo completo para trabajar con esos datos: descarga, descompresión, selección del píxel correspondiente a una estación, y comparación con registros del IMN.

## ¿Por qué existe este repositorio?

La comparación entre estimaciones satelitales y mediciones en tierra es una tarea común en hidrología y meteorología, pero requiere manejar formatos específicos (NetCDF), conversiones de zona horaria, y alineación temporal entre fuentes con distinta resolución.

Este repositorio automatiza ese flujo y documenta un caso de estudio con la estación Upala del IMN.

## Flujo de trabajo

```
1. descargador-hidroestimador.R     → descarga archivos .nc.gz desde AWS S3
2. descomprimir-hidroestimador.R    → descomprime a .nc en subcarpeta nc/
3. seleccionador-hidroestimador.R   → extrae el valor del pixel de la estacion
                                       y lo guarda en CSV con hora local
```

Cada script se ejecuta por separado, en orden. El rango de fechas se define en el descargador.

## Scripts

### `descargador-hidroestimador.R`

Descarga archivos del GHE desde el bucket público de NOAA en AWS S3.

- **Entrada:** rango de fechas (inicio, fin)
- **Salida:** archivos `.nc.gz` en la carpeta `descarga_Hidroestimador`

**Nota:** el descargador original apuntaba al FTP de NOAA (`ftp://ftp.star.nesdis.noaa.gov/...`), retirado en 2025. La versión actual apunta al bucket de AWS S3 donde NOAA migró los datos.

### `descomprimir-hidroestimador.R`

Descomprime los archivos `.nc.gz` y los guarda como `.nc` en la subcarpeta `nc/`.

- **Entrada:** archivos `.nc.gz`
- **Salida:** archivos `.nc` en `descarga_Hidroestimador/nc/`

**Nota:** `gunzip` elimina el `.gz` original después de descomprimir. Cada `.nc` pesa ~180 MB.

### `seleccionador-hidroestimador.R`

Extrae el valor de lluvia del píxel más cercano a una estación meteorológica, desde los archivos NetCDF.

- **Entrada:** carpeta `nc` con archivos `.nc`
- **Salida:** CSV con fecha, hora, lat, lon y valor de lluvia

**Nota:** el script está parametrizado con coordenadas de la estación Upala como ejemplo. Para usarlo con otra estación, solo hay que cambiar `lat_upala` y `lon_upala` por las coordenadas correspondientes. La fecha y hora se convierten de UTC a hora local de Costa Rica.

## Estructura del repositorio

```
.
├── README.md
├── descargador-hidroestimador.R
├── descomprimir-hidroestimador.R
├── seleccionador-hidroestimador.R
└── datos/
    └── comparacion_imn_ghe.csv
```

## Requisitos

- R (probado en versiones recientes)
- Paquetes:
  - `aws.s3`
  - `dplyr`
  - `R.utils`
  - `ncdf4`

Instalación:

```r
install.packages(c("aws.s3", "dplyr", "R.utils", "ncdf4"))
```

## Uso

1. Ajustar la ruta en `setwd()` de cada script.
2. Definir el rango de fechas en `descargador-hidroestimador.R`.
3. Ejecutar los scripts en orden.
4. Revisar el CSV resultante en la carpeta de trabajo.

## Caso de estudio

Se compararon datos del GHE con registros de la estación automática de Upala (IMN) para dos eventos:

| Fecha | Hora local | Lluvia IMN (mm) | Lluvia GHE (mm) |
|-------|------------|------------------|------------------|
| 2020-05-20 | 14:00 | 0 | 0 |
| 2020-05-20 | 15:00 | 79.8 | 0 |
| 2020-05-20 | 16:00 | 5.2 | 0 |
| 2021-08-21 | 15:00 | 0 | 0 |
| 2021-08-21 | 16:00 | 41.6 | 0 |
| 2021-08-21 | 17:00 | 0.6 | 0 |

### Observaciones

- El GHE no registró los eventos de lluvia puntual en la estación de Upala, aunque el IMN sí los registró.
- Al analizar el raster completo del GHE para el evento del 2020-05-20, se encontró que el producto sí detectó lluvia ese día en Costa Rica, pero concentrada en el Pacífico Sur (máximo de 56.8 mm en 8.29°N, -84.86°O).
- Esto es consistente con limitaciones conocidas del algoritmo del GHE en eventos de convección poco profunda o de corta duración.

### Conclusión preliminar

El Hidroestimador tiende a no registrar eventos de lluvia puntuales (una sola hora) en la estación de Upala, mientras que sí capta eventos de mayor duración o intensidad extendida. Se requieren más casos para confirmar el patrón.

## Contexto

Este repositorio forma parte de un portafolio de proyectos desarrollados para demostrar el uso de R aplicado a problemas de ingeniería y análisis de información ambiental.
