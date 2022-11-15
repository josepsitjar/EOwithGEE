****************************************************************************
Earth Observation with Google Earth Engine
****************************************************************************

1.1. Introduction
=================


Google Earth Engine is a cloud-based platform for scientific data analysis. It provides ready-to-use, cloud-hosted datasets and a large pool of servers.
One feature that makes Earth Engine particularly attractive is the ability to run large computations very fast by distributing them across a large pool of servers.
The ability to efficiently use cloud-hosted datasets and computation is enabled by the Earth Engine API.

GEE stores several image catalogs, including historical series that go back more than forty years, open available throught APIs.

Access to all this datasets, also with tools for analysis and visualization, make GEE a tool to be taken into account in the development of projects related to earth observation.


This activity is designed as an introduction to GEE running som simple scripts that will allow you to discover the platform potential.



1.2. Access to Google Earth Engine
==================================

An user Google account is required in order to acces GEE.
With this account you can register to the platform going to the initial page `GEE <https://earthengine.google.com/>`_ and then clicking to **Sign Up** button.

Once the GEE account is created, access to the `code editor <https://code.earthengine.google.com/>`_



1.2.1 GEE Code Editor
---------------------------------

The Code Editor is an integrated development environment for the Earth Engine JavaScript API. It offers an easy way to type, debug, run, and manage code. Also to visualize obtained results for code execution.

The Code Editor has the following elements


a) Javascript code editor
b) Map to visualize geospatial data
c) Drawing tools
d) API documentation
e) Script manager
f) Console output
g) Tasks tab


.. image:: static/codeeditor.png
  :width: 800
  :alt: Code editor


**Javascript code editor**: This is to create your own scripts. Just over this code editor, you'll find buttons to run, save and reset scripts, and also to generate a link to them.

**Map to visualize geospatial data**: The map is useful to visualize data and images from GEE catalog. Also to visualize your analisis results. Includes draw tools to create point, lines, polygon and rectangle geometries.

**API documentation**:  Contains the complete JavaScript API documentation. The documentation can be searched and browsed from the Docs tab.

**Script manager**: The Scripts tab is next to the API Docs in the left panel of the Code Editor. The Script Manager stores private, shared and example scripts in Git repositories hosted by Google. The repositories are arranged by access level, with your private scripts stored in a repository you own in the Owner folder.

**Output console**: When you print() something from your script, such as text, objects or charts, the result will be displayed in the Console. The console is interactive, so you can expand printed objects to get more details about them.

**Tasks tab**: Earth Engine Tasks are operations that are capable of running much longer than the standard API request timeout. These long-running tasks are the only mechanism for creating persistent artifacts in Earth Engine and adjacent systems


1.3. Search for a scene
========================

Knowing the identifier, we can look for a specific scene.

Now we are going to search a Landsat 8 image from Catalonia acquired on 2015-08-03.


.. code-block:: JavaScript

  // Image constructor instance:
	var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_198031_20150803');

	// Show the image over the map:
	Map.addLayer(image);




1.4. RGB band combination over a scene
==================================================

Create an RGB band combination using image bands:


.. code-block:: JavaScript

	// Image constructor instance:
	var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_198031_20150803');

	// RGB combination parameters
	// Max value is the maximum reflectivity value
	var visParams = {bands: ['B4', 'B3', 'B2'], max: 0.3};

	// Show the image over the map:
	Map.addLayer(image, visParams, 'true-color composite')


1.5 Búsqueda de una imagen si no conocemos su referencia concreta
==================================================================

En caso de no conocer la referencia concreta a una imagen, podemos hacer una búsqueda a partir de una colección y seleccionar aquella que coincida con una ubicación.
En primer lugar, tenemos que utilizar las herramientas de edición de GEE para digitalizar un punto y cambiarle el nombre (ej. point).

A continuación:


.. code-block:: JavaScript

	// Instanciamos la colección de Landsat 8
	var l8 = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA');

	// Aplicamos un filtro, relativo a la ubicación del punto
	var spatialFiltered = l8.filterBounds(point);

	// Aplicamos un filtro, relativo a un rango de fechas
	var temporalFiltered = spatialFiltered.filterDate('2021-12-01', '2021-12-31');

	// Ordenamos las escenas en función de la cobertura de nubes
	var sorted = temporalFiltered.sort('CLOUD_COVER');

	// Seleccionamos la primera escena
	var scene = sorted.first();

	// Añadimos la escena en el mapa
	Map.addLayer(scene, {}, 'default RGB');


1.6 Cargamos toda una colección de imágenes
===========================================

Podemos aprovechar para cargar todas las imagenes de la colección para crear un mosaico que cubra toda la superfície del mapa:

.. code-block:: JavaScript

	var l8 = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA');
	var landsat2016 = l8.filterDate('2016-01-01', '2016-12-31');
	var visParams = {bands: ['B4', 'B3', 'B2'], max: 0.3};

	Map.addLayer(landsat2016, visParams, 'l8 collection');


Este mosaico presenta un problema, y es que se visualizan escenas con muchas nubes, dado que por defecto se muestra el píxel mas reciente de todo el stack de imagenes.

Podemos modificar este comportamiento por defecto, indicando a GEE que tome el valor medio de todo el stack de píxeles de las imagenes de la colección (no el mas reciente). Se eliminarán nubes (valor mas alto de píxel) y sombras (valor mas bajo).Simplemente, añadiendo a la variable landsat2016 el filtro .median(): var landsat2016 = l8.filterDate('2016-01-01', '2016-12-31').median();

El *script* quedaría del siguiente modo:

.. code-block:: JavaScript

	var l8 = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA');
	var landsat2016 = l8.filterDate('2016-01-01', '2016-12-31').median();
	var visParams = {bands: ['B4', 'B3', 'B2'], max: 0.3};

	Map.addLayer(landsat2016, visParams, 'l8 collection');


1.7 Índices de vegetación
==========================

Volvemos a trabajar sobre una escena, y calculamos un índice, en este caso el índice de vegetación NDVI.

.. code-block:: JavaScript

	// Instanciamos la imagen con el constructor:
	var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_198031_20150803');

	// Calculamos el valor de NDVI.
	var nir = image.select('B5');
	var red = image.select('B4');
	var ndvi = nir.subtract(red).divide(nir.add(red)).rename('NDVI');

	var ndviParams = {min: -1, max: 1, palette: ['blue', 'white', 'green']};

	// Mostramos la imagen en el mapa:
	Map.addLayer(ndvi, ndviParams, 'NDVI image');


También se puede usar una función predefinida de GEE para el cálculo del NDVI:

.. code-block:: JavaScript

	var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_198031_20150803');

	// Utilizamos la función nomralizedDifference(A,B) para el cálculo del NDVI
	var ndvi = image.normalizedDifference(['B5', 'B4']);

  // Creamos la paleta de color
	var palette = ['FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718',

               '74A901', '66A000', '529400', '3E8601', '207401', '056201',

               '004C00', '023B01', '012E01', '011D01', '011301'];

	// Añadimos la capa al mapa
	Map.addLayer(ndvi, {min: 0, max: 1, palette: palette}, 'NDVI');


1.8 Sacar el máximo potencial de trabajar con un catálogo de datos en la nube
==============================================================================

Todo lo que hemos visto hasta ahora, lo podemos hacer de forma mas o menos fácil en un entoro SIG local. Pero el hecho de trabajar con una nuve de datos como las que ofrece GEE, es poder, por ejemplo, evaluar la evolución del NDVI, en un punto concreto, durante un período relativamente largo.

Debemos tener digitalizado un punto en GEE, i ejecutamos este script (en este caso, nos valemos de una función predefinida en GEE para el cálculo del NDVI)


.. code-block:: JavaScript

	// Importamos la colección LANDSAT 8 i filtramos para el año 2016
	var l8 = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA').filterDate('2016-01-01', '2016-12-31');

	// Aplicamos una función sobre la colección, para calcular la banda NDVI
	var withNDVI = l8.map(function(image) {
	var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');

	return image.addBands(ndvi);

	});

	// Creamos el gráfico
	var chart = ui.Chart.image.series({
	imageCollection: withNDVI.select('NDVI'),
	region: point,
	reducer: ee.Reducer.first(),
	scale: 30
	}).setOptions({title: 'NDVI over time'});

	// Mostramos el grafico en la consola
	print(chart);


1.9 Analizar la evolución de la LST (Land Surface Temperature)
================================================================

Otra ventaja de trabajar con un catálogo tan extenso de imágenes es la de poder analizar, por ejemplo, la evolución de la temperatura en superfície (LST).

Utilizaremos, para ello, la capa LST de Modis.


.. code-block:: JavaScript

	// En primer lugar, aplicamos una máscara sobre la zona de España
	// Creamos una máscara
	// Importamos una colección de datos con los límites de cada país
	var dataset = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017');

	// Aplicamos un filtro para seleccionar Spain
	var spainBorder = dataset.filter(ee.Filter.eq('country_na', 'Spain'));

	// Añadimos Spain al mapa
	Map.centerObject(spainBorder, 6);
	Map.addLayer(spainBorder);

	// A continuación, importamos los datos de temperatura (LST) del sensor MODIS
	// Importamos la colección LST de MODIS
	var modis = ee.ImageCollection('MODIS/MOD11A2');

	// Definimos el rango de datos. Fecha de inicio y final
	// Desede la fecha de inicio + un año
	var start = ee.Date('2017-01-01');
	var dateRange = ee.DateRange(start, start.advance(1, 'year'));

	// Aplicamos el filtro a la colección de datos MODIS para incorporar únicamente los datos de la fecha seleccionada
	var mod11a2 = modis.filterDate(dateRange);

	// Seleccionamos la banda LST a 1km
	var modLSTday = mod11a2.select('LST_Day_1km');

	// Convertir de grados Kelvin a Celsius
	// Aplicamos una función para convertir los datos de Kelvin a Celsius
	var modLSTc = modLSTday.map(function(img) {

	  return img

	    .multiply(0.02)

	    .subtract(273.15)

	    .copyProperties(img, ['system:time_start']);

	});

	// Creamos un gráfico con la evolución de la temperatura
	var ts1 = ui.Chart.image.series({
	  imageCollection: modLSTc,
	  region: spainBorder,
	  reducer: ee.Reducer.mean(),
	  scale: 1000,
	  xProperty: 'system:time_start'})
	  .setOptions({
	     title: 'LST 2015 Time Series',
	     vAxis: {title: 'LST Celsius'}});
	print(ts1);


1.10 Configurar el gráfico para visualizar la comparativa de LST entre diferentes años
=======================================================================================

.. code-block:: JavaScript

	// En primer lugar, aplicamos una máscara sobre la zona de España
	// Creamos una máscara
	// Importamos una colección de datos con los límites de cada país
	var dataset = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017');

	// Aplicamos un filtro para seleccionar Spain
	var spainBorder = dataset.filter(ee.Filter.eq('country_na', 'Spain'));

	// Añadimos Spain al mapa
	Map.centerObject(spainBorder, 6);
	Map.addLayer(spainBorder);

	// A continuación, importamos los datos de temperatura (LST) del sensor MODIS
	// Importamos la colección LST de MODIS
	var modis = ee.ImageCollection('MODIS/MOD11A2');

	// Definimos el rango de datos. Fecha de inicio y final
	// Desede la fecha de inicio + un año
	var start = ee.Date('2014-01-01');
	var dateRange = ee.DateRange(start, start.advance(2, 'year'));

	// Aplicamos el filtro a la colección de datos MODIS para incorporar únicamente los datos de la fecha seleccionada
	var mod11a2 = modis.filterDate(dateRange);

	// Seleccionamos la banda LST a 1km
	var modLSTday = mod11a2.select('LST_Day_1km');

	// Convertir de grados Kelvin a Celsius
	// Aplicamos una función para convertir los datos de Kelvin a Celsius
	var modLSTc = modLSTday.map(function(img) {
	return img
		.multiply(0.02)
		.subtract(273.15)
		.copyProperties(img, ['system:time_start']);
	});


	// Creamos un gráfico con la evolución de la temperatura
	var chart = ui.Chart.image.doySeriesByYear({
									imageCollection: modLSTc,
									bandName: 'LST_Day_1km',
									region: spainBorder,
									regionReducer: ee.Reducer.mean(),
									scale: 1000,
									})

	print(chart);
