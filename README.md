

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
  
## 3. Limpieza de datos, Identificar problemas, Controlar consistencia de los datos

- Data Profiling :	Analizar los datos para entender su estructura, contenido y calidad: Descriptive Statistics, Data Visualization, Data Auditing.
  Campos originales:
  * num_expediente : El número de expediente se refiere a un codigo que apunta a cada persona involucrada en un accidente de tráfico. Así por ejemplo un caso pdoría ser un accidente en que hay 3 personas involucradas, un motorista (conductor) , un turismo (conductor) y una tercera persona (pasajero) del vehiculo. Cada uno de estos registros debe concoordar con el resto de datos de su mismo num_expediente excepto los que refieren a la persona involucrada, sexo y rango de edad. El contenido de este campo no ha sido modificado. 
          
  * fecha	hora : Este campo contiene la fecha. El formato provisto era fecha-hora, hemos convertido el dato a solo fecha, ya que la hora que contenia este dato no contenia informacion en el 99% de los casos y además disponemos de un campo hora que veremos a continucacion. 
    
  * hora : Este campo contiene la hora. El mayor número de registro contenian un valor y era correcto.
     
  * localizacion : Este campo contiene el lugar del accidente, sea una calle con un número, una intersección, o el kilometro de una autovia. En principio no ha sido necesario modificar nada de este campo. 
    
  * tipo_accidente : Este campo estaba correcto un su mayor parte se limpió unos datos vacios, y otros que contenian datos del campo localizacion (errores). También se detectó que los datos de colisiones multiples y colisiones dobles tenían errores ya que el número de conductores involucrados no podia ser correcto (colisión doble con uno o ningún conductor, multiples con menos de 3 conductores,etc...)
    Esto se reparó desde la base de datos que creamos usando SQL.
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
    Los diferentes tipos incluyen :
    - Colisión doble: Accidente de tráfico ocurrido entre dos vehículos en movimiento, (colisión frontal, fronto lateral, lateral)
    - Colisión múltiple: Accidente de tráfico ocurrido entre más de dos vehículos en movimiento.
    - Alcance: Accidente que se produce cuando un vehículo circulando o detenido por las circunstancias del tráfico es golpeado en su parte posterior por otro vehículo.
    - Choque contra obstáculo o elemento de la vía: Accidente ocurrido entre un vehículo en movimiento con conductor y un objeto inmóvil que ocupa la vía o zona apartada de la misma, ya sea vehículo         estacionado, árbol, farola, etc.
    - Vuelco: Accidente sufrido por un vehículo con más de dos ruedas y que por alguna circunstancia sus neumáticos pierden el contacto con la calzada quedando apoyado sobre un costado o sobre el techo.
    - Caída: Se agrupan todas las caídas relacionadas con el desarrollo y las circunstancias del tráfico, (motocicleta, ciclomotor, bicicleta, viajero bus, etc.,).
    - Otras causas: Recoge los accidentes por atropello a animal, despeñamiento, salida de la vía, y otros		.	
  

  * estado_meteorológico : Se normalizo los datos de este campo poniendo valores unicos para referirnos al mismo tipo de fenómeno atmosférico ( Unificar la forma que esta escrito , mayuscula primera letra, y unificar la forma de describir el fenómeno , Granizando Granizo --> Granizo).
    
  * tipo_vehiculo : Se unifico los datos de este campo y un subtipo de vehiculo, se resumió en Turismo y Motocicleta (Son los valores que nos interesan para nuestra investigación)
    
  * tipo_persona : Este campo estaba mayormente correcto. Los valores validos Son Conductor, Pasajero o Peatón.
  * rango_edad : Este campo estaba correcto, contiene valores de edad en rangos.
  * sexo : Esta campo contiene H (hombre), o M (mujer).
  * lesividad : Ciertos campos información incorrecta. Los valores validos para este campo son :
        - Asistencia sanitaria ambulatorio con posteridad.
        - Asistencia sanitaria inmediata en centro de salud o mutua.
        - Asistencia sanitaria sólo en el lugar del accidente.
        - Atención en urgencias sin posterior ingreso.
        - Fallecido 24 horas.
        - Ingreso inferior o igual a 24 horas.
        - Ingreso superior a 24 horas.
        - Se desconoce.
        - Sin asistencia sanitaria.
    
  * coordenada_x_utm y coordenada_y_utm: Estas coordenadas se unificaron a un estandar de precision de tipo 123456.000 para posteriormente generar con ellos la longitud y la latitud. 
  * positiva_alcohol y positiva_droga Estos 2 campos contenian muchos valores vacios o nulos. Se conviertieron los valores S (Si) a 1 y el resto de valores a 0, para usar estos campos como tipo booleano.
 

4. Análisis: Transformación y Modelado	
Enumeración de los dasboards	
Variables compuestas	
5. Visualización del Dashboard	
6. Deploy del Dashboard	
7. Colaboración y Control de Versiones	
