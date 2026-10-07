# Pipeline ETL: exportaciones del NEA (1993-2024)

Trabajo Práctico Integrador, Unidad II. Diplomatura Universitaria en Data Analytics e Inteligencia Artificial Aplicada (UNNE).

Autor: Gonzalo Exequiel Fabaro Razongles

## Qué hace este pipeline

Descarga series de tiempo de exportaciones de Chaco, Corrientes, Formosa y Misiones desde la API pública de datos.gob.ar, las transforma en un dataset analítico y lo guarda en disco con controles de calidad y un registro de cada corrida.

Tiene tres etapas, cada una en su archivo:

| Archivo | Responsabilidad |
|---|---|
| `src/extract.py` | Descarga de la API y guarda el crudo en `data/raw/` |
| `src/transform.py` | Pasa de formato ancho a largo, calcula las columnas derivadas y une con los rubros |
| `src/load.py` | Valida con quality checks y guarda el CSV, el JSON de resumen y el log |

`src/main.py` orquesta las tres etapas y `config.py` concentra la configuración (rutas, IDs de series, parámetros).

## Cómo instalarlo y ejecutarlo

Requisitos: Python 3.8 o superior y Git. El proyecto usa solo la biblioteca estándar, así que no hay que instalar dependencias.

```
git clone https://github.com/gonzafabaro98-tech/TP-DIPLOMATURA-DA.git
cd TP-DIPLOMATURA-DA
python src/main.py
```

La primera corrida necesita internet para descargar los datos. Después se puede trabajar sin conexión con lo ya descargado:

```
python src/main.py --sin-internet
```

Para correr los tests:

```
python tests/test_transform.py
```

Las carpetas `data/` y `logs/` no se suben al repositorio (están en `.gitignore`): se generan al ejecutar el pipeline.

## De dónde salen los datos

API de Series de Tiempo del portal de datos abiertos del Estado argentino (datos.gob.ar), con datos del INDEC. No requiere credenciales.

- Dataset 357.1: exportaciones por provincia y país de destino.
- Dataset 350.1: exportaciones por provincia y rubro.
- Unidad: millones de dólares FOB. Período: 1993-2024.

Documentación de la API: https://datosgobar.github.io/series-tiempo-ar-api/

## Salidas

- `data/processed/exportaciones_nea.csv`: dataset final con 13 columnas y 1.408 filas (4 provincias x 11 destinos x 32 años).
- `data/processed/resumen.json`: ficha técnica con fuente, período, cantidad de filas, estadísticas básicas y resultado de los controles de calidad.
- `logs/pipeline.log`: una línea por cada corrida.

## Controles de calidad

Antes de guardar, el pipeline valida cuatro condiciones críticas que cortan el proceso si fallan: cantidad mínima de filas, columnas del contrato, ausencia de duplicados por (provincia, año, destino) y valores dentro de rango. Además emite una advertencia sobre la cobertura de los datos derivados.

## Qué encontré en los datos

En Chaco, en 2024, casi la mitad de las exportaciones (48,23%, unos 193,75 millones de dólares) corresponde a la categoría "Resto", que agrupa a los destinos que el INDEC no individualiza. El único país que se destaca es China, con el 27,61%. Entre los destinos menores hay variaciones muy grandes de un año a otro: Indonesia creció 93% respecto de 2023, mientras que Egipto cayó 53% y Colombia 67%. Esto muestra que los mercados chicos son muy volátiles y que las ventas de Chaco dependen de pocos compradores grandes.