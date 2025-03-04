## Obtención y unión de series de datos discretos climatológicos de estaciones terrestres
Keywords: `IDEAM` `Weather-station` `DHIME` `Rain` `Air-temperature` `Evaporation` `Water-flow` `Python` `Pandas` `os.path.isfile` `os.remove` `glob` `ZipFile` `os.rename` `pd.concat` `to_csv`

![R.LTWB](Graph/CNEStationDatasetDownload.png)

Para la creación de los mapas requeridos para la realización del balance hidrológico, es necesario a partir de las estaciones seleccionadas para la zona de estudio, obtener las series de valores discretos de precipitación total mensual, temperatura máxima diaria, temperatura mínima diaria y evaporación total diaria. Para la comparación de los caudales obtenidos, también son requeridas las series de caudal medio mensual.                       

<div align="center"><a href="http://www.youtube.com/watch?feature=player_embedded&v=9elIte7RYXg" target="_blank"><img src="../../.icons/R.LTWB_PlayVideo.svg" alt="R.LTWB" width="240" border="0" /></a><sub><br>https://www.youtube.com/watch?v=9elIte7RYXg<br>Playlist: https://youtube.com/playlist?list=PLZGvAjHkhphDKXvnhkp0oQb22EHWVd0W8</sub><br><br></div>


### Objetivos

* A partir de las estaciones identificadas y seleccionadas para la zona de estudio, obtener las series o registros de las estaciones a partir de los datos disponibles en el portal DHIME del IDEAM - Colombia.
* Utilizando Python y la librería Pandas, integrar los archivos de datos comprimidos descargados que contienen archivos de texto separados por comas, en un único archivo .csv.


### Requerimientos

* [Microsoft Excel from Office 64 bits.](https://aka.ms/office-install) 
* [Python 3+](https://www.python.org/)
* [Pandas para Python 3+](https://pandas.pydata.org/)
* [Glosario de variables IDEAM.](../../.datasets/GlosarioVariables.xlsx) [:mortar_board:Aprender.](../CNEStation)
* [Listado de estaciones seleccionadas de precipitación en la zona de estudio.](../../.datasets/CNE_IDEAM_OE_ZE_Precipitacion.dbf) [:mortar_board:Aprender.](../CNEStation)
* [Listado de estaciones seleccionadas de temperatura del aire en la zona de estudio.](../../.datasets/CNE_IDEAM_OE_ZE_TemperaturaAire.dbf) [:mortar_board:Aprender.](../CNEStation)
* [Listado de estaciones seleccionadas de evaporación potencial en la zona de estudio.](../../.datasets/CNE_IDEAM_OE_ZE_Evaporacion.dbf) [:mortar_board:Aprender.](../CNEStation)
* [Listado de estaciones seleccionadas de caudal en la zona de estudio.](../../.datasets/CNE_IDEAM_OE_ZE_NivelCaudal.dbf) [:mortar_board:Aprender.](../CNEStation)


### Diagrama general de procesos

El siguiente diagrama representa los procesos generales requeridos para el desarrollo de esta actividad.

<div align="center">
<br><img alt="R.LTWB" src="Graph/CNEStationDatasetDownload.svg" width="75%"><br>
<sub>Convenciones generales en diagramas: clases de entidad en azul, dataset en gris oscuro, grillas en color verde, geo-procesos en rojo y procesos manuales en amarillo. Líneas con guiones corresponden a procedimientos opcionales.</sub><br><br>
</div>


### Conceptos generales 


#### Glosario de variables IDEAM

El libro de Excel del [glosario de variables del IDEAM - Colombia](../../.datasets/GlosarioVariables.xlsx), se compone de 3 hojas de cálculo que contienen el listado de etiquetas básicas de los diferentes parámetros de la red hidroclimatológica y las etiquetas de las series diarias derivadas que corresponden a datos que se calculan a partir de las series básicas. La versión utilizada para ejemplificar esta clase corresponde a la fecha 2022.07.31. 


#### Catálogo de datos de los registros discretos IDEAM

Tomados directamente de los archivos de texto separados por comas obtenidos del servicio [DHIME](http://dhime.ideam.gov.co/atencionciudadano/) del [IDEAM](http://www.ideam.gov.co/) y tipos devueltos por Python / Pandas.

| Atributo         | Tipo       | Descripción                                                                                                                                                                                                                                    |
|------------------|------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CodigoEstacion   | int64      | Código de la estación. CodigoEsta en archivos dBase.                                                                                                                                                                                           |
| NombreEstacion   | object     | Nombre de la estación. Incluye el código de la estación entre corchetes. NombreEsta en archivos dBase.                                                                                                                                         |
| Latitud          | float64    | Latitud en grados decimales.                                                                                                                                                                                                                   |
| Longitud         | float64    | Longitud en grados decimales.                                                                                                                                                                                                                  |
| Altitud          | int64      | Altitud o cota sobre el nivel del mar en metros.                                                                                                                                                                                               |
| Categoria        | object     | Categoría de la estación: Pluviométrica, Limnimétrica, Limnigráfica, Climática Ordinaria, Climática Principal, Pluviográfica, Meteorológica Especial, Agrometeorológica, Sinóptica Principal, Radio Sonda, Mareográfica, Sinóptica Secundaria. |
| Entidad          | object     | Entidad encargada.                                                                                                                                                                                                                             |
| AreaOperativa    | object     | Área operativa que administra la estación. AreaOperat en archivos dBase.                                                                                                                                                                       |
| Departamento     | object     | Departamento o zonificación política. Equivalente a estados en otros países. Departamen en archivos dBase.                                                                                                                                     |
| Municipio        | object     | Municipio o subzonificación política. Equivalente a condado en otros países.                                                                                                                                                                   |
| FechaInstalacion | datetime64 | Fecha de instalación. FechaInsta en archivos dBase.                                                                                                                                                                                            |
| FechaSuspension  | datetime64 | Fecha de suspensión. FechaSuspe en archivos dBase.                                                                                                                                                                                             |
| IdParametro      | object     | Nombre del parámetro hidro-climatológico. Ver Glosario de variables IDEAM en servicio DHIME en la pestaña Recursos. IdParametr en archivos dBase.                                                                                              |
| Etiqueta         | object     | Etiqueta del parámetro que depende si es una variable básica o derivada. Ver Glosario de variables IDEAM en servicio DHIME en la pestaña Recursos.                                                                                             |
| DescripcionSerie | object     | Descripción de la etiqueta del parámetro que indica el nombre del parámetro el estadístico y la frecuencia. Ver Glosario de variables IDEAM en servicio DHIME en la pestaña Recursos. Descripcio en archivos dBase.                            |
| Frecuencia       | object     | Frecuencia de captura o de cálculo. Ver Glosario de variables IDEAM en servicio DHIME en la pestaña Recursos.                                                                                                                                  |
| Fecha            | datetime64 | Fecha de registro.                                                                                                                                                                                                                             |
| Valor            | float64    | Valor registrado.                                                                                                                                                                                                                              |
| Grado            | int64      | Grado del dato asignado por el IDEAM dentro de sus procesos de validación, p. ej. -1, 4, 5, 50.                                                                                                                                                |
| Calificador      | object     | Calificador del dato registrado, p. ej. Dudoso, Est. Interpolación, Est. Otros métodos, Est. Regresión, Estimado, Río Seco, Sección Inestable. Calificado en archivos dBase.                                                                   |
| NivelAprobacion  | int64      | Nivel de aprobación. NivelAprob en archivos dBase.                                                                                                                                                                                             |


#### Nivel de aprobación de cada dato[^1]

| Código | Nivel de aprobación |
|:------:|:-------------------:|
|  900   |     Preliminar      |
|  1100  |     En revisión     |
|  1200  |     Definitivo      |


> La información validada o definitiva al encontrarse certificada, ha surtido el proceso de validación técnica necesaria que garantiza la calidad del dato y determina la oficialidad de la información que podrá ser utilizada para toma de decisiones. Para el desarrollo del caso de estudio, usaremos la información IDEAM en todos los niveles de aprobación disponibles.


### Procedimiento general manual

1. Dentro de la carpeta _D:\R.LTWB\\.datasets_, cree una nueva carpeta con el nombre _IDEAM_, y un nuevo libro de Microsoft Excel con el nombre _CNEStationDatasetDownload.xlsx_. Dentro del libro de Excel, crear 4 hojas y nombrar como: `Precipitacion`, `TemperaturaAire`, `EvaporacionPotencial` y `Caudal`.

2. Desde Excel, abra los siguientes archivos de estaciones seleccionadas que fueron exportados a formato DBase File .dbf, copie y pegue todos los registros a las hojas del libro de Excel:

<div align="center">

| Parámetro             | Archivo .dbf estaciones                                                                    | Hoja Excel           | Estaciones |
|-----------------------|--------------------------------------------------------------------------------------------|----------------------|------------|
| Precipitación         | [CNE_IDEAM_OE_ZE_Precipitacion.dbf](../../.datasets/CNE_IDEAM_OE_ZE_Precipitacion.dbf)     | Precipitacion        | 139        |
| Temperatura del aire  | [CNE_IDEAM_OE_ZE_TemperaturaAire.dbf](../../.datasets/CNE_IDEAM_OE_ZE_TemperaturaAire.dbf) | TemperaturaAire      | 42         |
| Evaporación potencial | [CNE_IDEAM_OE_ZE_Evaporacion.dbf](../../.datasets/CNE_IDEAM_OE_ZE_Evaporacion.dbf)         | EvaporacionPotencial | 41         |
| Caudal                | [CNE_IDEAM_OE_ZE_NivelCaudal.dbf](../../.datasets/CNE_IDEAM_OE_ZE_NivelCaudal.dbf)         | Caudal               | 65         |

</div>

3. En cada hoja del libro de Excel, agregue las siguientes dos columnas:

<div align="center">

| Columna | Descripción                                                                                                    |
|---------|----------------------------------------------------------------------------------------------------------------|
| Ready   | Para marcación de cuales estaciones con series de datos descargables se encuentran en el servicio DHIME. `Y/N` |
| File    | Nombre del archivo descargado.                                                                                 |

</div>

> El propósito de estas columnas es registrar para cuáles estaciones fue posible realizar la descarga de series y en que archivo de la secuencia de descarga se encuentran los datos obtenidos. Posteriormente, efectuaremos la lectura de los mismos parámetros a partir de información satelital, pero únicamente para las estaciones terrestres que disponen de información.

4. En cada hoja, mueva las columnas `DEPARTAMEN` y `MUNICIPIO` después de la columna `CODIGO`. En el menú _Data_, seleccione la opción _Filter_ y desde el menú _View_, congele la primera fila correspondiente a las etiquetas de columna. Ordene las estaciones por Departamento y código para facilitar la búsqueda y la descarga.

![R.LTWB](Screenshot/MicrosoftExcelCNEStationDatasetDownload.png)

> Debido a que los datos se han copiado desde archivos .dbf, es posible que por la codificación de texto, no se visualicen correctamente las tildes y eñes.

5. Ingrese al portal [DHIME](http://dhime.ideam.gov.co/atencionciudadano/) del [IDEAM - Colombia](http://www.ideam.gov.co/), acepte los términos de uso y de clic en _Aceptar_.

![R.LTWB](Screenshot/DHIMETerminos.png)


### Descarga de series de precipitación

6. En la pestaña _Consultar_, defina los siguientes parámetros:

* Periodo
  * Fecha Inicial: `31/12/1899`, correspondiente a fecha a partir de la cual se pueden obtener registros desde el servicio DHIME.
  * Fecha Final: `01/01/2022`, para la obtención de series utilizaremos años cronológicos completos cuyo último registro corresponde al 31 de diciembre de cada año. Dentro del servicio DHIME, es necesario incluir el 01 de enero del año inmediatamente siguiente debido a que el proceso de filtrado se realiza para valores menores qué. 
* Serie de Tiempo y Frecuencia: `Estándar`, debido a que la descarga a realizar corresponde a series de datos mensuales para los datos de precipitación.
* Parámetro: `PRECIPITACIÓN`.
* Variable: `Lista Completa` seleccionando `Precipitación total mensual` que de acuerdo con el Glosario de Variables del IDEAM, corresponde a una _variable derivada_ debido a que requiere de un proceso de cálculo a partir de la sumatoria de los valores registrados horarios, diarios o de frecuencias inferiores. 

> Las descargas a partir de la definición de la fecha final, también pueden ser realizadas a partir de [años hidrológicos](https://es.wikipedia.org/wiki/A%C3%B1o_hidrol%C3%B3gico) que pueden corresponder a periodos del 01 de junio al 31 de mayo del año inmediatamente siguiente, fracciones de invierno a verano o ciclos estacionales dependiendo de la zona geográfica.

![R.LTWB](Screenshot/DHIMEPrecipitacionParametros.png)

Glosario de variables del IDEAM - Series básicas
![R.LTWB](Screenshot/MicrosoftExcelGlosarioVariablesBasicasPrecipitacion.png)

Glosario de variables del IDEAM - Series derivadas
![R.LTWB](Screenshot/MicrosoftExcelGlosarioVariablesDerivadasPrecipitacion.png)

7. En la sección _Datos Estación_ de DHIME, seleccione el departamento de Bolívar y todos sus municipios. Desde el archivo de Excel y la hoja _Precipitacion_, seleccione el código de la primera estación y oprima las teclas <kbd>Ctrl</kbd> + <kbd>c</kbd>; en el navegador de Internet oprima <kbd>Ctrl</kbd> + <kbd>f</kbd> para abrir el cuadro de búsqueda, con <kbd>Ctrl</kbd> + <kbd>v</kbd> pegue el código y de <kbd>Enter</kbd>. Automáticamente será dirigido a la estación, marque la casilla de selección ubicada en la parte izquierda y repita el procedimiento hasta marcar 10 estaciones si estas se encuentran en el mismo Departamento. En el libro de Excel, ingrese `Y` en la columna `Ready` si la estación se encuentra disponible en DHIME y `N` si no aparece en la búsqueda.

> Para navegadores de Internet en español oprima <kbd>Ctrl</kbd> + <kbd>b</kbd> para realizar búsquedas.
> 
> DHIME permite realizar descargas simultáneas de registros para máximo 10 estaciones.

![R.LTWB](Screenshot/DHIMEPrecipitacionDescarga.png)

Debido a que para el departamento de Bolívar solo descargaremos los datos de 7 estaciones, es necesario dar clic en el botón `Agregar a la Consulta` y luego dar clic en el botón `Agregar Otros` que permitirá agregar 3 estaciones más sin perder el periodo definido. Es necesario volver a seleccionar manualmente el parámetro para continuar la búsqueda y marcado de las estaciones.

![R.LTWB](Screenshot/DHIMEPrecipitacionDescargaQueryAdd.png)

Seleccione y agregue las 3 primeras estaciones del departamento del Cesar registradas en el libro de Excel para completar el bloque de 10 estaciones. De clic en el botón `Agregar a la Consulta` que lo llevará a la pestaña de descargas y de clic en el botón `Descargar`. Obtendrá en la carpeta de descargas de su sistema operativo, un archivo comprimido en formato .zip con el nombre _datos.zip_ que contendrá dentro un archivo de texto separado por comas denominado _excel.csv.csv_. En el libro de Excel, registre el nombre del archivo descargado en la columna `File`.

![R.LTWB](Screenshot/DHIMEPrecipitacionDescargaCSV.png)

> Dependiendo del tipo de frecuencia, de la longitud de la serie y del número de solicitudes simultáneas realizadas por otros usuarios al servidor DHIME, la descarga de cada archivo comprimido podrá tardar algunos segundos o minutos.

En el portal DHIME, de clic en el botón `Limpiar` y repita el procedimiento de descarga anterior hasta obtener los registros de precipitación de todas las estaciones requeridas para la zona de estudio. Recuerde registrar los valores correspondientes en las columnas `Ready` y `File` del libro de Excel.

Resumen de datos obtenidos para precipitación total mensual
* 139 seleccionadas, 131 descargables de IDEAM
* Comprimidos de datos.zip a datos (14).zip


### Descarga de series de temperatura del aire

8. Para la descarga de las series temporales de temperatura, utilice los siguientes parámetros:

* Periodo
  * Fecha Inicial: `31/12/1899`, correspondiente a fecha a partir de la cual se pueden obtener registros desde el servicio DHIME.
  * Fecha Final: `01/01/2022`, para la obtención de series utilizaremos años cronológicos completos cuyo último registro corresponde al 31 de diciembre de cada año. Dentro del servicio DHIME, es necesario incluir el 01 de enero del año inmediatamente siguiente debido a que el proceso de filtrado se realiza para valores menores qué. 
* Serie de Tiempo y Frecuencia: `Estándar`, debido a que la descarga a realizar corresponde a series de datos diarios.
* Parámetro: `TEMPERATURA`.
* Variable: `Lista Completa` seleccionando y descargando primero `Temperatura máxima diaria` y luego `Temperatura mínima diaria`, que de acuerdo con el Glosario de Variables del IDEAM, corresponden a _variables básicas_ debido a que no requieren de un proceso de cálculo y son obtenidas directamente de un registrador. 

> DHIME no dispone de registros derivados de temperatura media mensual calculados a partir de los datos diarios, por lo que es necesario calcular los valores medios a partir de los datos máximos y mínimos.

![R.LTWB](Screenshot/DHIMETemperaturaAireParametros.png)

Glosario de variables del IDEAM - Series básicas
![R.LTWB](Screenshot/MicrosoftExcelGlosarioVariablesBasicasTemperaturaAire.png)

Glosario de variables del IDEAM - Series derivadas
![R.LTWB](Screenshot/MicrosoftExcelGlosarioVariablesDerivadasTemperaturaAire.png)

Resumen de datos obtenidos para temperatura máxima diaria
* 42 seleccionadas, 25 descargables de IDEAM
* Comprimidos de datos (15).zip a datos (17).zip

Resumen de datos obtenidos para temperatura mínima diaria
* 42 seleccionadas, 25 descargables de IDEAM
* Comprimidos de datos (18).zip a datos (20).zip


### Descarga de series de evaporación potencial

9. Para la descarga de las series temporales de evaporación potencial, utilice los siguientes parámetros:

* Periodo
  * Fecha Inicial: `31/12/1899`, correspondiente a fecha a partir de la cual se pueden obtener registros desde el servicio DHIME.
  * Fecha Final: `01/01/2022`, para la obtención de series utilizaremos años cronológicos completos cuyo último registro corresponde al 31 de diciembre de cada año. Dentro del servicio DHIME, es necesario incluir el 01 de enero del año inmediatamente siguiente debido a que el proceso de filtrado se realiza para valores menores qué. 
* Serie de Tiempo y Frecuencia: `Estándar`, debido a que la descarga a realizar corresponde a series de datos diarios.
* Parámetro: `EVAPORACIÓN`.
* Variable: `Lista Completa` seleccionando y descargando `Evaporación total diaria`, que de acuerdo con el Glosario de Variables del IDEAM, corresponde a una _variable básica_ debido a que no requieren de un proceso de cálculo y es obtenida directamente de un registrador. 

![R.LTWB](Screenshot/DHIMEEvaporacionParametros.png)

Glosario de variables del IDEAM - Series básicas
![R.LTWB](Screenshot/MicrosoftExcelGlosarioVariablesBasicasEvaporacion.png)

Glosario de variables del IDEAM - Series derivadas
![R.LTWB](Screenshot/MicrosoftExcelGlosarioVariablesDerivadasEvaporacion.png)

Resumen de datos obtenidos para evaporación total diaria
* 41 seleccionadas, 1 descargable de IDEAM correspondiente a la estación 29065130
* Comprimido de datos datos (21).zip

>Como puede observar en las imágenes de referencia, al seleccionar este parámetro no es posible descargar los datos derivados correspondientes a evaporación total mensual. 


### Descarga de series de caudal medio mensual

10. Para la descarga de las series temporales de caudal medio mensual, utilice los siguientes parámetros:

* Periodo
  * Fecha Inicial: `31/12/1899`, correspondiente a fecha a partir de la cual se pueden obtener registros desde el servicio DHIME.
  * Fecha Final: `01/01/2022`, para la obtención de series utilizaremos años cronológicos completos cuyo último registro corresponde al 31 de diciembre de cada año. Dentro del servicio DHIME, es necesario incluir el 01 de enero del año inmediatamente siguiente debido a que el proceso de filtrado se realiza para valores menores qué. 
* Serie de Tiempo y Frecuencia: `Estándar`, debido a que la descarga a realizar corresponde a series de datos medios mensuales.
* Parámetro: `CAUDAL`.
* Variable: `Lista Completa` seleccionando y descargando `Caudal medio mensual`, que de acuerdo con el Glosario de Variables del IDEAM, corresponde a una _variable derivada_ debido a que requiere de un proceso de cálculo a partir del promedio de los valores registrados a nivel horario. 

![R.LTWB](Screenshot/DHIMECaudalParametros.png)

Glosario de variables del IDEAM - Series básicas
![R.LTWB](Screenshot/MicrosoftExcelGlosarioVariablesBasicasCaudal.png)

Glosario de variables del IDEAM - Series derivadas
![R.LTWB](Screenshot/MicrosoftExcelGlosarioVariablesDerivadasCaudal.png)

Resumen de datos obtenidos para caudal medio mensual
* 65 seleccionadas, 57 descargables de IDEAM
* Comprimidos de datos (22).zip a datos (27).zip

Al finalizar la descarga de todos los registros para todos los parámetros requeridos, copie los archivos comprimidos en la carpeta _[D:\R.LTWB\\.datasets\IDEAM](../../.datasets/IDEAM)_.

![R.LTWB](Screenshot/DatasetsIDEAM.png)


### Unión de series descargadas utilizando Python y Pandas

Diagrama de flujo

<div align="center">
<br><img alt="R.LTWB" src="Graph/CNEStationCSVJoin.svg" width="50%"><br>
<sub>Convenciones generales en diagramas: clases de entidad en azul, dataset en gris oscuro, grillas en color verde, geo-procesos en rojo, procesos automáticos o semiautomáticos en guiones rojos y procesos manuales en amarillo. Líneas conectoras con guiones corresponden a procedimientos opcionales.</sub><br><br>
</div>

Para optimizar los procesos posteriores de exploración y análisis de datos ([Exploratory Data Analysis - EDA](https://towardsdatascience.com/exploratory-data-analysis-8fc1cb20fd15)), es necesario integrar todos los registros obtenidos para los diferentes parámetros de las estaciones seleccionadas para la zona de estudio. Para este proceso utilizaremos Python y la librería Pandas a través del siguiente script localizado en la carpeta _.src_.

Para la ejecución del script, previamente se requiere de la instalación de Python 3+ y la librería Pandas. [:mortar_board:Aprender.](../../Section01/Requirement)

Script [D:\R.LTWB\.src\CNEStationCSVJoin.py](../../.src/CNEStationCSVJoin.py)
```
# -*- coding: UTF-8 -*-
# Name: CNEStationCSVJoin.py
# Description: this script uncompress and join multiple .zip files get manually from http://dhime.ideam.gov.co/atencionciudadano/ into a single .csv file.
# Repository: ../CNEStationDatasetDownload
# License: https://github.com/rcfdtools/R.LTWB/wiki/License
# Requirements: Python 3+, Pandas

# Libraries
import glob
from zipfile import ZipFile
import os
import pandas as pd

# Procedure
path = 'D:/R.LTWB/.datasets/IDEAM/' # Your local .zip files path, use ../.datasets/IDEAM/ for relative path
join_file = 'IDEAMJoined.csv' # Joined file name
if os.path.isfile(path + join_file):
    os.remove(path + join_file)
zip_files = glob.glob(path + 'datos*.zip')
for i in zip_files:
    print('Unzipping %s' %i)
    ZipFile(i).extractall(path)
    os.rename(path + 'excel.csv.csv', i+'.csv')
csv_files = glob.glob(path + 'datos*.csv')
df = pd.concat(map(pd.read_csv, csv_files), ignore_index=True)
df.to_csv(path + join_file, encoding='utf-8', index=False)
print(df)
for csv_file in csv_files:
    os.remove(csv_file)
```

> Para la ejecución correcta del script `CNEStationCSVJoin.py`, en la carpeta `.datasets/IDEAM/` no deben existir archivos comprimidos .zip diferentes a los descargados desde el servicio DHIME que inicien con el nombre _datos_ y los comprimidos originalmente obtenidos sí deberán iniciar como _datos****.zip_.

1. Utilizando un editor de texto (p. ej. Notepad o Notepad++), abra el script y defina en la variable `path` la ruta o el directorio de volcado, p. ej. `path = '../.datasets/IDEAM/'`, que corresponde a la ruta relativa o `path = 'D:/R.LTWB/.datasets/IDEAM/'` que corresponde a la ruta absoluta donde se encuentran los archivos .zip descargados desde el servicio DHIME de IDEAM.

2. En Microsoft Windows, ejecute el _Command Prompt_ o _CMD_, ingrese `D:` y de <kbd>Enter</kbd> para cambiar a la unidad D:\ donde se encuentra el repositorio R.LTWB. Utilizando el comando  `CD D:\R.LTWB\.datasets\IDEAM` diríjase a la carpeta donde están contenidos los datos descargados. Usando el comando `DIR /W`, liste en una vista ancha, los archivos contenidos en la carpeta IDEAM y verifique que se encuentren los archivos comprimidos .zip obtenidos previamente.

![R.LTWB](Screenshot/Windows11CMDDir.png)

3. Limpie la consola con el comando `CLS` y utilizando en el `CMD` la instrucción `C:\Python3.10.5\python.exe "D:\R.LTWB\.src\CNEStationCSVJoin.py"`, ejecute el script que realizará la descompresión e integración de los archivos y creará o actualizará el archivo `IDEAMJoined.csv`.

![R.LTWB](Screenshot/Windows11PythonCNEStationCSVJoin1.png)
![R.LTWB](Screenshot/Windows11PythonCNEStationCSVJoin2.png)

Como puede observar, se han integrado 514927 registros en el archivo [IDEAMJoined.csv](../../.datasets/IDEAM/IDEAMJoined.zip) que tiene un tamaño aproximado de 154 MB.

4. Desde en bloc de notas _Notepad++_, abra el archivo IDEAMJoined.csv y verifique el total de registros obtenidos.

![R.LTWB](Screenshot/NotepadIDEAMJoined.png)

A partir de este momento dispone de registros integrados de diferentes variables hidroclimatológicas.


### Actividades complementarias:pencil2:

En la siguiente tabla se listan las actividades complementarias que deben ser desarrolladas y documentadas por el estudiante en un único archivo de Adobe Acrobat .pdf. El documento debe incluir portada (mostrar nombre completo, código y enlace a su cuenta de GitHub), numeración de páginas, tabla de contenido, lista de tablas, lista de ilustraciones, introducción, objetivo general, capítulos por cada ítem solicitado, conclusiones y referencias bibliográficas.

| Actividad | Alcance                                                                                                                                                                                                                                                                                                                                         |
|:---------:|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     1     | Para las estaciones identificadas en la actividad complementaria [CNEStation](../CNEStation) relacionadas con brillo solar, radiación solar, humedad del aire cerca al suelo y parámetros relacionados con viento y nubosidad, descargue los registros disponibles en DHIME del IDEAM e integre las series a las presentadas en esta actividad. | 
|     2     | En caso que no pueda descargar los registros de estaciones a partir del servicio DHIME del IDEAM - Colombia, busque el servicio de descarga de cada entidad y realice la descarga de los datos requeridos de las estaciones identificadas dentro de su zona de estudio.                                                                         | 


### Referencias

* http://dhime.ideam.gov.co/atencionciudadano/
* http://www.ideam.gov.co/
* https://es.wikipedia.org/wiki/A%C3%B1o_hidrol%C3%B3gico
* https://stackoverflow.com/questions/3207219/how-do-i-list-all-files-of-a-directory
* https://stackoverflow.com/questions/3451111/unzipping-files-in-python
* https://stackoverflow.com/questions/2491222/how-to-rename-a-file-using-python
* https://www.tutorialspoint.com/how-to-merge-all-csv-files-into-a-single-dataframe-python-pandas
* https://stackoverflow.com/questions/16923281/writing-a-pandas-dataframe-to-csv-file
* https://stackoverflow.com/questions/32834731/how-to-delete-a-file-by-extension-in-python
* https://towardsdatascience.com/exploratory-data-analysis-8fc1cb20fd15
* [Histórico de series hidrometeorológicas CAR Cundinamarca - Colombia](https://www.car.gov.co/vercontenido/2524)
* [Descarga de datos Sistema de alerta de Bogotá - Colombia](https://www.sire.gov.co/web/sab/informacion-hidrometeorologica)


### Control de versiones

| Versión    | Descripción                                                                                                                                                                                              | Autor                                      | Horas |
|------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------|:-----:|
| 2023.02.07 | Guión, audio, video, edición y publicación.                                                                                                                                                              | [rcfdtools](https://github.com/rcfdtools)  |  2.5  |
| 2022.08.17 | Complementación documentación para unión de series descargadas utilizando Python y Pandas. Diagrama de procesos CNEStationDatasetDownloadFlowchart.svg, diagrama de flujo CNEStationCSVJoinFlowchart.svg | [rcfdtools](https://github.com/rcfdtools)  |   4   |
| 2022.08.16 | \.src\CNEStationCSVJoin.py: creación de script en Python para descomprimir y unir múltiples archivos .csv contenidos dentro de archivos .zip, en un único archivo .csv.                                  | [rcfdtools](https://github.com/rcfdtools)  |   2   |
| 2022.08.15 | Versión inicial. Descarga completa de series para estaciones de la zona de estudio desde el portal DHIME                                                                                                 | [rcfdtools](https://github.com/rcfdtools)  |   7   |


##

_R.LTWB es de uso libre para fines académicos, conoce nuestra licencia, cláusulas, condiciones de uso y como referenciar los contenidos publicados en este repositorio, dando [clic aquí](https://github.com/rcfdtools/R.LTWB/wiki/License)._

_¡Encontraste útil este repositorio!, apoya su difusión marcando este repositorio con una ⭐ o síguenos dando clic en el botón Follow de [rcfdtools](https://github.com/rcfdtools) en GitHub._

| [Anterior](../CNEStationElevation) | [:house: Inicio](../../Readme.md) | [:beginner: Ayuda / Colabora](https://github.com/rcfdtools/R.LTWB/discussions/16) | [Siguiente](../RemoteSensing) |
|------------------------------------|-----------------------------------|-----------------------------------------------------------------------------------|-------------------------------|

[^1]: http://dhime.ideam.gov.co/atencionciudadano/