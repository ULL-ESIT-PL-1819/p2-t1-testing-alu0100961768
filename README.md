# P2-T1: Transforming Data and Testing Continuously:
[![Build Status](https://travis-ci.org/ULL-ESIT-PL-1819/p2-t1-testing-alu0100961768.svg?branch=master)](https://travis-ci.org/ULL-ESIT-PL-1819/p2-t1-testing-alu0100961768)

En esta práctica tendremos como objetivo transformar datos en formato XML a LDJ (Line-delimited JSON), usando JSON como intermediario, mediante NodeJS.

## Objetivos

- Trabajar extayendo y parseando datos de ficheros.
- Trabajar con Gulp y Gulpfiles para automatizar trabajos manuales.
- Desarrollar pruebas unitarias para testear nuestro código usando mocha y chai.
- Usar control de versiones con herramientas como Travis.


## Entorno de trabajo

<b>Framework utilizado: </b>
[Visual Studio Code](https://code.visualstudio.com)

## Paso a paso

Comenzaremos esta práctica con dos ficheros 'data' (que contendrá la info a palo seco con la que trabajaremos) y 'database', donde trabajaremos.

Lo primero de todo será descargar todos los datos en formato .rdf en 'data'.
 
```
curl -O http://www.gutenberg.org/cache/epub/feeds/rdf-files.tar.bz2
tar -xvjf rdf-files.tar.bz2
```

Nuestro objetivo es poder trabajar con la GID, el título, la lista de autores y la lista de temas de cada libro, que se encuentran, junto con otra información, en los archivos .rdf. Pero antes de ponernos manos a la obra, crearemos test utilizando Mocha y Chai, basandonos en la filosofial de desarrollo guiado por el comportamiento (BDD).

Lo primero que haremos será instalar mocha y chai.

``` 
npm init -y
npm install --save-dev --save-exact mocha@2.4.5 chai@3.5.0
```
Deberíamos editar nuestro ```package.json``` para que ejecute nuestros tests con mocha. Editar la seccion de 'scripts' y añadiendo ```"test": "mocha"``` es lo único que necesitamos hacer.

Ahora crearemos nuestro archivo de tests, donde podemos crear un simple test en Chai, como puede ser el siguiente:

```
​ 	​'use strict'​;
​ 	
​ 	​const​ fs = require(​'fs'​);
​ 	​const​ expect = require(​'chai'​).expect;
​ 	​const​ rdf = fs.readFileSync(​`​${__dirname}​/pg132.rdf`​);
​ 	describe(​'parseRDF'​, () => {
​ 	  it(​'should be a function'​, () => {
​ 	    expect(parseRDF).to.be.a(​'function'​);
​ 	  });
```

Ahora, cuando ejecutemos nuestros tests con ```npm test```, estos, evidentemente, fallaran (¡Aún no existe 'parse-rdf'!)

Si creamos el fichero lib y declaramos parse-rdf, nuestro test funcionará. Desarrollar poco a poco los test, y a continuación el código para la pequeña tarea que estamos interesados en resolver es la base de BDD.

Usando ```npm run test:watch```, podemos monitorizar cosntantemente que test han tenido éxito y que tests no.
AL partir de ahora, se entiende que iremos creando tests a medida que encesitemos desarrollar una nueva funcionalidad.

Para transformar información de rdf. a JSON, usaremos Cheerio, un parser de NodeJS para archivos XML.

Nuestro parser para .rdf puede terminar siendo algo así
```nodejs
//parse-rdf.js
	​'use strict'​;
​ 	​const​ cheerio = require(​'cheerio'​);
​ 	
​ 	module.exports = rdf => {
​ 	  ​const​ $ = cheerio.load(rdf);
​ 	
​ 	  ​const​ book = {};
​ 	
​ 	  book.id = +$(​'pgterms​​\\​​:ebook'​).attr(​'rdf:about'​).replace(​'ebooks/'​, ​''​);
​ 	
​ 	  book.title = $(​'dcterms​​\\​​:title'​).text();
​ 	
​ 	  book.authors = $(​'pgterms​​\\​​:agent pgterms​​\\​​:name'​)
​ 	    .toArray().map(elem => $(elem).text());
​ 	  book.subjects = $(​'[rdf​​\\​​:resource$="/LCSH"]'​)
​ 	    .parent().find(​'rdf​​\\​​:value'​)
​ 	    .toArray().map(elem => $(elem).text());
​ 	
​ 	  ​return​ book;
​ 	};
```

Ahora necesitamos transformarlo en .json.
```nodejs
//rdf-to-json
	​#!/usr/bin/env node​
​ 	​const​ fs = require(​'fs'​);
​ 	​const​ parseRDF = require(​'./lib/parse-rdf.js'​);
​ 	​const​ rdf = fs.readFileSync(process.argv[2]);
​ 	​const​ book = parseRDF(rdf);
​ 	console.log(JSON.stringify(book, ​null​, ​'  '​));
```

Este pequeño módulo puede convertir contenido .rdf en .json. Ahora, podemons modificar nuestro ```parse-rdf.js``` para obtener la info extra necesaria que se nos pide en las modificaciones.

Finalmente, podemos crear, de forma opcional, un ```gulpfile``` que ejecute lost test y el programa por nosotros, sin necesidad de estar ejecutandolo manualemnte.

Un gulpfile para un proyecto como este puede parecerse a este:

```gulpfile
var gulp = require('gulp');
var shell = require('gulp-shell');

gulp.task("c5-get-guttenberg", shell.task(

	'npm test &&' +
	'node rdf-to-json.js test/pg132.rdf' 
));
```
Que nos permitiría ejecutar el programa para el fichero ```pg132.rdf```.
Tambien podemos, de forma opcional, añadir integración continua con Travis, mostrando en el README si las pruebas de nuestro proyecto estan siendo superadas o no en cada versión. En este caso, se puede ver que nuestros test estan superando las pruebas, como indica la badge al comienzo del readme.

### Alumno
<b>Nombre:</b> Germán Alfonso Teixidó.

<b>Alu:</b> alu0100961768.
