# Quiz SQL 2 — Bienes TIC para Enseñanza (DANE / MEN)

Repositorio con los scripts SQL y resultados del Quiz SQL 2, desarrollado sobre los microdatos del **Ministerio de Educación Nacional (MEN)** relacionados con el uso de bienes TIC en sedes educativas de Colombia para los años 2022 y 2023.

---


## 🗃️ Dataset

| Año  | Fuente | Enlace |
|------|--------|--------|
| 2022 | DANE / MEN — Microdatos | https://microdatos.dane.gov.co/index.php/catalog/801/get-microdata |
| 2023 | DANE / MEN — Microdatos | https://microdatos.dane.gov.co/index.php/catalog/834 |

### Estructura de las tablas (`tic_2022` y `tic_2023`)

| Campo             | Tipo       | Descripción                                      |
|-------------------|------------|--------------------------------------------------|
| `sede_codigo`     | TEXT       | Código único de la sede educativa                |
| `periodo_id`      | INTEGER    | ID interno del período de reporte                |
| `periodo_anio`    | INTEGER    | Año del reporte (2022 o 2023)                    |
| `actividad_id`    | INTEGER    | ID numérico de la actividad TIC                  |
| `actividad_codigo`| TEXT       | Código alfanumérico de la actividad ('04','05','07') |
| `actividad_nombre`| TEXT       | Descripción completa de la actividad pedagógica  |

### Actividades TIC registradas

| Código | Nombre |
|--------|--------|
| 04 | Exposición y enseñanza de los contenidos curriculares en red (intranet) |
| 05 | Consulta de contenidos pedagógicos, mediante buscadores en internet |
| 07 | Aprendizaje y evaluación del aprendizaje utilizando la plataforma virtual o multimedia digital |

---

## 🛠️ Herramienta utilizada

[SQLite Online](https://sqliteonline.com/) — ambas tablas fueron importadas como demo SQLite con **Column Name: First Line**.

---

## 📝 Queries y resultados

### Punto 1 — Reconocimiento de `tic_2023`

Exploración inicial de la tabla para conocer su contenido y los valores únicos de actividad.

```sql
-- Consulta A: Todos los registros de tic_2023
SELECT * FROM tic_2023;

-- Consulta B: Valores distintos en actividad_nombre
SELECT DISTINCT actividad_nombre
FROM tic_2023;
```

**Resultados:** `Resultado_1_punto_1A.csv` · `Resultado_2_punto_1B.csv`

---

### Punto 2 — Exploración básica de `tic_2022`

Reporte de reconocimiento con alias de columnas, ordenado por sede y limitado a 50 registros.

```sql
SELECT
    sede_codigo      AS codigo_sede,
    periodo_anio     AS anio_reporte,
    actividad_codigo AS cod_actividad,
    actividad_nombre AS nombre_actividad
FROM tic_2022
ORDER BY sede_codigo ASC
LIMIT 50;
```

**Resultado:** `Resultado_punto_2.csv`

---

### Punto 3 — Creación de tabla `tic_sedes_resumen`

Diseño de tabla nueva para consolidar el número de actividades TIC por sede y año, con restricción UNIQUE sobre `sede_codigo + anio`.

```sql
CREATE TABLE tic_sedes_resumen (
    resumen_id        INTEGER PRIMARY KEY AUTOINCREMENT,
    sede_codigo       INTEGER NOT NULL,
    anio              INTEGER NOT NULL,
    Total_actividades INTEGER,
    Tiene_internet    BOOLEAN,
    Fecha_carga       DATETIME CURRENT_TIMESTAMP,
    CONSTRAINT uq_sede_anio UNIQUE (sede_codigo, anio)
);
```

> Este punto es DDL (definición de estructura), no genera un resultado CSV.

---

### Punto 4 — Sedes con actividad '05' por departamento (2023)

Conteo de sedes únicas por departamento que registraron consulta de contenidos por internet, mostrando solo los departamentos con más de 500 sedes.

```sql
SELECT
    SUBSTR(sede_codigo, 1, 2)       AS cod_departamento,
    COUNT(DISTINCT sede_codigo)     AS total_sedes_unicas
FROM tic_2023
WHERE actividad_codigo = '05'
GROUP BY SUBSTR(sede_codigo, 1, 2)
HAVING COUNT(DISTINCT sede_codigo) > 500
ORDER BY total_sedes_unicas DESC;
```

**Resultado:** `Resultado_punto_4.csv` — 21 departamentos cumplen el filtro.

---

### Punto 5 — Comparación 2022 vs 2023 por sede (INNER JOIN)

Para cada sede presente en ambos años, se calcula el total de actividades TIC registradas en cada período y se determina si hubo crecimiento, decrecimiento o si se mantuvo igual.

```sql
SELECT
    a.sede_codigo                          AS codigo_sede,
    count(a.actividad_id) AS total_act_2022,
    count(b.actividas_id) AS total_act_2023,
    total_act_2023 - total_act_2022   AS diferencia,
    CASE
        WHEN b.total_act_2023 - a.total_act_2022 > 0 THEN 'CRECIÓ'
        WHEN b.total_act_2023 - a.total_act_2022 < 0 THEN 'DECRECIÓ'
        ELSE 'SIN CAMBIO'
    END AS tendencia
FROM
    (SELECT sede_codigo, COUNT(*) AS total_act_2022 FROM tic_2022 GROUP BY sede_codigo) a
INNER JOIN
    (SELECT sede_codigo, COUNT(*) AS total_act_2023 FROM tic_2023 GROUP BY sede_codigo) b
    ON a.sede_codigo = b.sede_codigo
WHERE a.total_act_2022 >= 2
ORDER BY diferencia DESC
LIMIT 30;
```

**Resultado:** `Resultado_punto_5.csv` — Top 30 sedes con al menos 2 registros en 2022, ordenadas por mayor crecimiento.

---

## ⚙️ Cómo reproducir los resultados

1. Ir a [sqliteonline.com](https://sqliteonline.com/) y seleccionar **SQLite** como demo.
2. Importar los dos archivos CSV originales asegurándose de que **Column Name** diga **First Line**.
3. Nombrar las tablas `tic_2022` y `tic_2023` respectivamente.
4. Copiar y ejecutar los queries del archivo `queries_quiz_sql2.sql` punto por punto.
