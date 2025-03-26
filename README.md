

# AlDaLa Proyect - A Data Analisis learning project.

<img src="logo_aldala.png"> 

## 1. Definición del Objetivo del Dashboard	

El objetivo es analizar estos datos para ajustar primas y diseñar estrategias de prevención basadas en patrones de siniestralidad.
- ¿Qué problema queremos resolver?
    Analisis perfilado para la Politica de precios en base a criterios estadísticos. 
- ¿Quién usará el dashboard?
    Analistas y directivos
- ¿Qué métricas (KPIs) serán claves?
    Variables de riesgo, correlaciones entre variables, perfiles.
- ¿Qué preguntas debe responder el dashboard?
    Me permite entender lo anterior y consultar en función de perfiles

## 2. Importar los datos	

  Los datos se importaron a google sheet en primer momento para realizar un analisis de los mismos y evaluar los puntos a seguir, debido al volumen de datos el cual era muy grande para tratarlos con google sheets (muy lento, el software se bloqueaba), se decidió crear una base de datos usando mysql para poder interactuar y limpiar los datos de forma ágil. 
  
## 3. Data Cleaning Process	

- Data Profiling :	Analizar los datos para entender su estructura, contenido y calidad: Descriptive Statistics, Data Visualization, Data Auditing.
  Campos originales:
  * num_expediente : El número de expediente se refiere a un codigo que apunta a cada persona involucrada en un accidente de tráfico. Así por ejemplo un caso pdoría ser un accidente en que hay 3 personas involucradas, un motorista (conductor) , un turismo (conductor) y una tercera persona (pasajero) del vehiculo. Cada uno de estos registros debe concoordar con el resto de datos de su mismo num_expediente excepto los que refieren a la persona involucrada, sexo y rango de edad. El contenido de este campo no ha sido modificado. 
          
  * fecha	hora : Este campo contiene la fecha. El formato provisto era fecha-hora, hemos convertido el dato a solo fecha, ya que la hora que contenia este dato no contenia informacion en el 99% de los casos y además disponemos de un campo hora que veremos a continucacion. 
    
  * hora : Este campo contiene la hora. El mayor número de registro contenian un valor y era correcto.
     
  * localizacion
  * tipo_accidente
  * estado_meteorológico
  * tipo_vehiculo
  * tipo_persona
  * rango_edad
  * sexo
  * lesividad
  * coordenada_x_utm
  * coordenada_y_utm
  * positiva_alcohol
  * positiva_droga


2. Identifying Issues	Detectar problemas de calidad de los datos, como valores faltantes, duplicados e inconsistencias.
3. Handling Missing Values	Rellenar los valores faltantes utilizando técnicas como la imputación o la eliminación.
4. Removing Duplicates	Identifying and removing duplicate records.
5. Correcting Inconsistencies	Estandarizar los datos para asegurar la coherencia.
6. Validating Data	Verificar la precisión y la integridad de los datos.
7. Documenting Changes	Mantener un registro de los cambios realizados durante el proceso de limpieza de datos.
4. Análisis: Transformación y Modelado	
Enumeración de los dasboards	
Variables compuestas	
5. Visualización del Dashboard	
6. Deploy del Dashboard	
7. Colaboración y Control de Versiones	
