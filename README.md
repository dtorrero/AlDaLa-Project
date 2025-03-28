# AlDaLa Proyect - A Data Analisis learning project.

<img src="logo_aldala.png"> 

Estrellas invitadas: David, Laura y Ale. 

## 1. Definición del Objetivo del Dashboard

A partir de los datos de accidentes en la zona norte de Madrid durante 2020 y 2021, se realizará un estudio de riesgo destinado a su uso por parte de una aseguradora.

El objetivo es analizar estos datos para ajustar primas y diseñar estrategias de prevención basadas en patrones de siniestralidad.

- **¿Qué problema queremos resolver?**  
  Mejorar la capacidad de toma de decisiones en relación con la política de precios, basándonos en criterios estadísticos.

- **¿Quién usará el dashboard?**  
  Analistas y directivos.

- **¿Qué métricas (KPIs) serán clave?**  
  Variables de riesgo, correlaciones entre variables, perfiles.

- **¿Qué preguntas debe responder el dashboard?**  
  Debe permitir comprender los patrones anteriores y realizar consultas en función de distintos perfiles.

---

## 2. Importación de los Datos

Los datos se importaron inicialmente a Google Sheets para realizar un análisis preliminar y definir los siguientes pasos.  
Sin embargo, debido al gran volumen de datos, el tratamiento en esta herramienta resultó complejo (muy lenta, el software se bloqueaba).

En paralelo, se decidió crear una base de datos en **MySQL** para poder interactuar con los datos y limpiarlos de forma ágil y eficiente.

---
## 3. Exploración Inicial: ¿Qué nos encontramos?

Antes de comenzar con la limpieza y el modelado, realizamos una primera exploración del conjunto de datos usando técnicas de métricas de estadística descriptiva y gráficos. Estos fueron los principales hallazgos:

- **Gran volumen de datos:**  
  El dataset contiene un número elevado de registros, lo que hizo necesario usar herramientas más robustas que Google Sheets para su tratamiento.

- **Formatos variados:**  
  Algunos campos presentaban formatos inconsistentes (por ejemplo, fechas y horas en diferentes estilos), lo que dificultaba su análisis directo.

- **Presencia significativa de valores en blanco o nulos:**  
  Muchos campos estaban incompletos o contenían valores vacíos, lo cual obligó a tomar decisiones sobre imputación o exclusión.

- **Alta variabilidad en los valores:**  
  Se encontraron múltiples formas de expresar una misma categoría (por ejemplo, variaciones en nombres de vehículos, fenómenos meteorológicos o tipos de accidente), lo que requirió una normalización exhaustiva.

Esta etapa fue clave para diseñar una estrategia de limpieza y estructuración de los datos, permitiendo un análisis más preciso y confiable en las siguientes fases del proyecto.

---

## 4. Limpieza de Datos, Identificación de Problemas y Control de Consistencia

### Campos originales:

- **`num_expediente`**  
  Corresponde a un código que identifica a cada persona involucrada en un accidente de tráfico.  
  Por ejemplo, un mismo accidente puede involucrar a tres personas: un motorista (conductor), un turismo (conductor) y un pasajero.  
  Todos los registros asociados al mismo `num_expediente` deben coincidir en la información general del accidente, excepto en los campos específicos de la persona (sexo, rango de edad, etc.). Este campo no fue modificado.

- **`fecha_hora`**  
  El campo original contenía tanto la fecha como la hora. Se transformó para conservar únicamente la fecha, ya que en el 99 % de los casos la información horaria no era relevante.  
  Además, el archivo incluía un campo específico de hora que se utilizó en su lugar.

- **`hora`**  
  Este campo contiene la hora del accidente. En la mayoría de los registros, el valor era correcto y útil.

- **`localización`**  
  Contiene el lugar del accidente (una calle con número, intersección o kilómetro de una autovía).  
  No fue necesario modificar este campo, ya que se utilizó principalmente la información de coordenadas, también presente en el archivo.

- **`tipo_accidente`**  
  En su mayoría, este campo estaba correcto. Se eliminaron registros vacíos y otros que contenían datos erróneos provenientes del campo `localización`.  
  Además, se detectaron inconsistencias en los tipos de colisión: por ejemplo, "colisión doble" con un solo conductor o "colisión múltiple" con menos de tres conductores.  
  Estas inconsistencias se corrigieron desde la base de datos utilizando SQL.

    ```
    DELETE FROM accidentes
    WHERE num_expediente IN (
    SELECT num_expediente
    FROM (
        SELECT 
            num_expediente,
            tipo_accidente,
            COUNT(DISTINCT CASE WHEN tipo_persona = 'conductor' THEN id END) AS conductor_count
        FROM 
            accidentes
        GROUP BY 
            num_expediente, 
            tipo_accidente
        HAVING 
            (tipo_accidente = 'colisión doble' AND COUNT(DISTINCT CASE WHEN tipo_persona = 'conductor' THEN id END) != 2) OR
            (tipo_accidente = 'colisión multiple' AND COUNT(DISTINCT CASE WHEN tipo_persona = 'conductor' THEN id END) < 3)
    ) AS problematic_cases
    );
    ```
### Tipos de accidentes

Los diferentes tipos de accidentes incluidos en el análisis son:

- **Colisión doble:**  
  Accidente de tráfico entre dos vehículos en movimiento (colisión frontal, fronto-lateral, lateral).

- **Colisión múltiple:**  
  Accidente de tráfico entre más de dos vehículos en movimiento.

- **Alcance:**  
  Accidente que se produce cuando un vehículo, ya sea circulando o detenido por las circunstancias del tráfico, es golpeado en su parte posterior por otro vehículo.

- **Choque contra obstáculo o elemento de la vía:**  
  Accidente entre un vehículo en movimiento y un objeto inmóvil que ocupa la vía o su entorno (vehículo estacionado, árbol, farola, etc.).

- **Vuelco:**  
  Accidente en el que un vehículo con más de dos ruedas pierde el contacto de los neumáticos con la calzada y queda apoyado sobre un costado o el techo.

- **Caída:**  
  Se agrupan todas las caídas relacionadas con el tráfico, como motocicletas, ciclomotores, bicicletas o pasajeros de autobús.

- **Otras causas:**  
  Incluye atropello a animal, despeñamiento, salida de la vía, entre otros.

---

### Resto de campos tratados

- **`estado_meteorológico`**  
  Se normalizaron los valores de este campo para unificar la forma de escritura (primera letra en mayúscula) y la descripción del fenómeno.  
  Por ejemplo: "Granizando", "Granizo" → **Granizo**

- **`tipo_vehiculo`**  
  Se unificaron los datos en dos grandes categorías relevantes para la investigación: **Turismo** y **Motocicleta**.

- **`tipo_persona`**  
  Este campo estaba mayormente correcto. Los valores válidos son: **Conductor**, **Pasajero** o **Peatón**.

- **`rango_edad`**  
  Este campo estaba correcto y contiene valores de edad agrupados por rangos.

- **`sexo`**  
  Este campo contiene los valores **H** (hombre) y **M** (mujer).

- **`lesividad`**  
  Algunos registros presentaban información incorrecta. Los valores válidos para este campo son:

  - Asistencia sanitaria ambulatoria con posterioridad  
  - Asistencia sanitaria inmediata en centro de salud o mutua  
  - Asistencia sanitaria solo en el lugar del accidente  
  - Atención en urgencias sin posterior ingreso  
  - Fallecido en las primeras 24 horas  
  - Ingreso inferior o igual a 24 horas  
  - Ingreso superior a 24 horas  
  - Se desconoce  
  - Sin asistencia sanitaria

- **`coordenada_x_utm` y `coordenada_y_utm`**  
  Se unificó su formato a un estándar de precisión tipo `123456.000`.  
  A partir de estas coordenadas, se generaron los valores de **longitud** y **latitud** utilizando una función desarrollada en AppScript llamada `UTMtoLatLon`.

```
/**
 * Convierte coordenadas UTM a Latitud y Longitud (WGS84).
 * @param {number} easting Coordenada X.
 * @param {number} northing Coordenada Y.
 * @param {number} zone Zona UTM (ej. 30).
 * @param {string} hemisphere "N" o "S".
 * @return {[number, number]} Latitud y Longitud en grados decimales.
 */
function UTMtoLatLon(easting, northing, zone, hemisphere) {
  // Asegurar formatos válidos
  easting = Number(easting);
  northing = Number(northing);
  zone = Number(zone);
  hemisphere = String(hemisphere).toUpperCase();

  if (isNaN(easting) || isNaN(northing) || isNaN(zone) || (hemisphere !== 'N' && hemisphere !== 'S')) {
    return ['Error', 'Parámetros inválidos'];
  }

  const a = 6378137.0;
  const e = 0.0818191908;
  const e1sq = 0.006739497;
  const k0 = 0.9996;

  const x = easting - 500000.0;
  const y = (hemisphere === 'S') ? northing - 10000000.0 : northing;

  const longOrigin = (zone - 1) * 6 - 180 + 3;

  const M = y / k0;
  const mu = M / (a * (1 - Math.pow(e,2)/4 - 3*Math.pow(e,4)/64 - 5*Math.pow(e,6)/256));

  const e1 = (1 - Math.sqrt(1 - e*e)) / (1 + Math.sqrt(1 - e*e));
  const J1 = (3*e1/2 - 27*Math.pow(e1,3)/32);
  const J2 = (21*Math.pow(e1,2)/16 - 55*Math.pow(e1,4)/32);
  const J3 = (151*Math.pow(e1,3)/96);
  const J4 = (1097*Math.pow(e1,4)/512);

  const fp = mu + J1*Math.sin(2*mu) + J2*Math.sin(4*mu) + J3*Math.sin(6*mu) + J4*Math.sin(8*mu);

  const sin_fp = Math.sin(fp);
  const cos_fp = Math.cos(fp);
  const tan_fp = Math.tan(fp);

  const C1 = e1sq * Math.pow(cos_fp,2);
  const T1 = Math.pow(tan_fp,2);
  const R1 = a * (1 - Math.pow(e,2)) / Math.pow(1 - Math.pow(e*sin_fp,2), 1.5);
  const N1 = a / Math.sqrt(1 - Math.pow(e*sin_fp,2));
  const D = x / (N1 * k0);

  const lat = fp - (N1 * tan_fp / R1) *
    (Math.pow(D,2)/2 - (5 + 3*T1 + 10*C1 - 4*Math.pow(C1,2) - 9*e1sq) * Math.pow(D,4)/24 +
    (61 + 90*T1 + 298*C1 + 45*Math.pow(T1,2) - 252*e1sq - 3*Math.pow(C1,2)) * Math.pow(D,6)/720);

  const lon = (D - (1 + 2*T1 + C1) * Math.pow(D,3)/6 +
    (5 - 2*C1 + 28*T1 - 3*Math.pow(C1,2) + 8*e1sq + 24*Math.pow(T1,2)) * Math.pow(D,5)/120) / cos_fp;

  const latDeg = lat * (180 / Math.PI);
  const lonDeg = (longOrigin + lon * (180 / Math.PI));

  return [latDeg, lonDeg];
}
```
- **`positiva_alcohol`** y **`positiva_droga`**  
  Ambos campos contenían numerosos valores vacíos o nulos.  
  Se transformaron los valores **S** (sí) a `1` y el resto de valores a `0`, para poder utilizarlos como campos booleanos en los análisis.

---

### Nuevos campos

- **`ID`**  
  Se añadió un nuevo campo `ID` como identificador único de cada registro, facilitando el manejo y trazabilidad de los datos.

---

## 5. Análisis: Transformación y Modelado

- **Enumeración de los dashboards:**  
  Se desarrollaron diferentes visualizaciones para representar de forma clara los datos clave para la aseguradora, facilitando el análisis por parte de los usuarios.

- **Variables compuestas:**  
  Se crearon variables derivadas y agrupaciones de datos que permiten identificar perfiles de riesgo, zonas críticas y patrones de siniestralidad más complejos.

---

## 6. Visualización del Dashboard

Se diseñó un dashboard interactivo orientado a analistas y directivos, con filtros por perfil, zona, tipo de accidente, entre otros.  
Incluye KPIs visuales y gráficos dinámicos para facilitar la exploración de los datos y la toma de decisiones.

---

## 7. Deploy del Dashboard

El dashboard fue desplegado en una plataforma accesible para los usuarios de la aseguradora.  
(🔧 *Aquí puedes detallar si usaste herramientas como Streamlit, Power BI, Tableau, etc., y si está disponible públicamente o solo internamente*).

---

## 8. Colaboración y Control de Versiones

El proyecto se gestiona mediante **Git** y **GitHub**, lo que facilita el trabajo colaborativo y el seguimiento de cambios.  
Se recomienda trabajar con ramas (`branches`) para nuevas funcionalidades y usar `pull requests` para revisión de código antes de fusionar en la rama principal.

---


