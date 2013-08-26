# Buenos Aires API Standards

* [Guidelines](#guidelines)
* [Pragmatic REST](#pragmatic-rest)
* [RESTful URLs](#restful-urls)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [Gestión de errores](#Gestión-de-errores)
* [Versiones](#versiones)
* [Record limits](#record-limits)
* [Request & Response Ejemplos](#request-response-Ejemplos)
* [Mock Responses](#mock-responses)
* [JSONP](#jsonp)

## Guidelines

Este documento provee guidelines y ejemplos para Buenos Aires API, buscamos fomentar la coherencia, mantenimiento y mejores practicas en las aplicaciones. BA API tiene por objetivo crear una interfase RESTful API para una buena experiencia de desarrollo.

En este documento se inspira en gran medida de:
* [WHITE HOUSE API](https://github.com/WhiteHouse/api-standards)
* [Designing HTTP Interfaces and RESTful Web Services](http://munich2012.drupal.org/program/sessions/designing-http-interfaces-and-restful-web-services)
* API Facade Pattern, by Brian Mulloy, Apigee
* Web API Design, by Brian Mulloy, Apigee
* [Fieldings Dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)

## Pragmatic REST

Esta guia tiene por objetivo darle soporte a una RESTful API. Salvo algunas excepciones:
* Agregar el numero de version de API en la URL (ver ejemplo abajo). No aceptar ningun request que no especifique un numero de version.
* Permitir a los usuarios solicitar formatos como JSON o XML de la siguiente manera:
    * http://buenosaires.gob.ar/api/v1/informe.json
    * http://buenosaires.gob.ar/api/v1/informe.xml

## RESTful URLs

### Guidelines generales para RESTful URLs
* Una URL identifica un recurso.
* URLs deben incluir nombres, no verbos.
* Utilice nombres en plural sólo por coherencia (sin nombres en singular).
* Utilice HTTP verbs (GET, POST, PUT, DELETE) para operar en las colecciones y elementos..
* Usted no necesita ir más allá de recurso/identificador/recurso.
* Ponga el número de versión en la base de la URL, por ejemplo http://buenosaires.gob.ar/v1/path/to/recurso.
* URL v. header:
    * Si cambia la logica para administrar la respuesta, agregarlo en la URL.
    * Si no modifica la logica para cada respuesta, como OAuth info, agregarlo en el Header.
* Especificar campos opcionales en una lista separada por comas.
* Los Formatos deben estar en la forma de api/v2/recurso/{id}.json

### Ejemplos de un Correcto uso de las URL
* Lista de Informes:
    * http://www.buenosaires.gob.ar/api/v1/informes.json
* Filtrando una query:
    * http://www.buenosaires.gob.ar/api/v1/informes.json?year=2011&sort=desc
    * http://www.buenosaires.gob.ar/api/v1/informes.json?topic=economy&year=2011
* Un unico informe en formato JSON:
    * http://www.buenosaires.gob.ar/api/v1/informes/1234.json
* Todos los articulos en (o pertenecen a) este informe:
    * http://www.buenosaires.gob.ar/api/v1/informes/1234/articles.json
* Todos los articulos de este informe en formato XML:
    * GET http://buenosaires.gob.ar/api/v1/informes/1234/articles.xml
* Especificar campos opcionales en una lista separada por comas:
    * http://www.buenosaires.gob.ar/api/v1/informes/1234.json?fields=titulo,subtitulo,fecha
* Agregar un articulo a un informe:
    * POST http://buenosaires.gob.ar/api/v1/informes/1234/articles

### Usos NO CORRECTOS de URL
* Nombres en singular:
    * http://www.buenosaires.gob.ar/informe
    * http://www.buenosaires.gob.ar/informe/1234
    * http://www.buenosaires.gob.ar/publicador/informe/1234
* Verbo en URL:
    * http://www.buenosaires.gob.ar/informe/1234/crear
* Filtro fuera de la query de consulta
    * http://www.buenosaires.gob.ar/informes/2011/desc

## HTTP Verbs

| HTTP METHOD | POST            | GET       | PUT         | DELETE |
| ----------- | --------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE          | READ      | UPDATE     | DELETE |
| /perros       | Create nuevo perro | List perros | Bulk update | Delete all perros |
| /perros/1234  | Error           | Show perro   | If exists, update perro; If not, error | Delete perro |


## Respuestas

* No incluir valores en las keys
* No terminos internos-especificos (ej. "node" y "taxonomy")
* La Metadata solo debe incluir propiedades directas de la respuesta.

### Ejemplos Correctos

No incluir valores en las keys:

    "tags": [
      {"id": "125", "nombre": "ambiente"},
      {"id": "834", "nombre": "calidad agua"}
    ],


### Ejemplos Incorrectos

Valores en las keys:

    "tags": [
      {"125": "ambiente"},
      {"834": "calidad agua"}
    ],


## Gestión de errores

Las respuestas de error deben incluir un código de estado HTTP, un mensaje común para el desarrollador, mensaje para el usuario final (en caso de ser necesario), el código de error interno (que corresponde a un número de error específico determinado internombrente), enlaces donde los desarrolladores pueden encontrar más info. Por ejemplo:

    {
      "status" : "400",
      "developerMessage" : "Descriptiva, una descripción clara del problema. Proporcionar desarrolladores sugerencias sobre cómo resolver sus problemas",
      "userMessage" : "De ser necesario, este mensaje puede darse al usuario final.",
      "errorCode" : "444444",
      "more info" : "http://www.buenosaires.gob.ar/developer/path/to/help/for/444444,
       http://drupal.org/node/444444",
    }

Utilice tres códigos de respuesta simples, comunes indicando (1) el éxito, (2) el fracaso debido a un problema del lado del cliente (client-side), (3) el fracaso debido a un problema en el servidor (server-side).
* 200 - OK
* 400 - Bad Request
* 500 - Internal Server Error


## Versiones

* Nunca lanzar una API sin un numero de version.
* Los numeros de version deben ser enteros, no decimales y con el prefijo 'v. Por ejemplo:
    * Bien: v1, v2, v3
    * Mal: v-1.1, v1.2, 1.3
* Mantener APIs de una version anterior al menos.


## Registros - Limites

* Si no se especifica un limite, el resultado se muestra con el limite por defecto.
* Para obtener 50 registros entre 75 hacer lo siguiente:
    * http://buenosaires.gob.ar/informes?limit=25&offset=50
    * offset=50 significa, ‘empezar con el registro cincuenta’
    * limit=25 significa, ‘devolver 25 registros’

La informacion acerca del limite de cantidad de registros tambien deberia ser incluida en los ejemplos de respuestas.

    {
        "metadata": {
            "resultset": {
                "count": 50,
                "offset": 25,
                "limit": 25
            }
        },
        "results": [
            { .. }
        ]
    }

## Ejemplos de Request & Response

### API Recursos

  - [GET /informes](#get-informes)
  - [GET /informes/[id]](#get-informesid)
  - [POST /informes/[id]/articulos](#post-informesidarticulos)

### GET /informes

Ejemplo: http://buenosaires.gob.ar/api/v1/informes.json

    {
        "metadata": {
            "resultset": {
                "count": 123,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "id": "1234",
                "type": "informe",
                "titulo": "Sistema de Agua Publico",
                "tags": [
                    {"id": "125", "nombre": "Ambiente"},
                    {"id": "834", "nombre": "Calidad Agua"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "informe",
                "titulo": "Escuelas Publicas",
                "tags": [
                    {"id": "125", "nombre": "Primaria"},
                    {"id": "834", "nombre": "ENET Nº4"}
                ],
                "created": "126251302"
            }
            {
                "id": 2351,
                "type": "informe",
                "titulo": "Escuelas Publicas",
                "tags": [
                    {"id": "125", "nombre": "Preescolar"},
                ],
                "created": "126251302"
            }
        ]
    }

### GET /informes/[id]

Ejemplo: http://buenosaires.gob.ar/api/v1/informes/[id].json

    {
        "id": "1234",
        "type": "informe",
        "titulo": "Sistema de Agua Publico",
        "tags": [
            {"id": "125", "nombre": "Ambiente"},
            {"id": "834", "nombre": "Calidad Agua"}
        ],
        "created": "1231621302"
    }



### POST /informes/[id]/articles

Ejemplo: Create – POST  http://buenosaires.gob.ar/api/v1/informes/[id]/articles

    {
        "titulo": "Viajes en Bicicleta",
        "autor_nombre": "Juan",
        "autor_apellido": "Perez",
        "autor_email": "Juan.Perez@buenosaires.gob.ar",
        "año": "2012"
        "mes": "Agosto"
        "dia": "18"
        "textot": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam eget ante ut augue scelerisque ornare. Aliquam tempus rhoncus quam vel luctus. Sed scelerisque fermentum fringilla. Suspendisse tincidunt nisl a metus feugiat vitae vestibulum enim vulputate. Quisque vehicula dictum elit, vitae cursus libero auctor sed. Vestibulum fermentum elementum nunc. Proin aliquam erat in turpis vehicula sit amet tristique lorem blandit. Nam augue est, bibendum et ultrices non, interdum in est. Quisque gravida orci lobortis... "

    }


## Simulacion Responses
Se sugiere que cada recurso acepta un parámetro 'simulacro' en el servidor de prueba. El envio de este parámetro debe devolver una respuesta de datos simulada (sin pasar por el servidor).

Implementar esta funcionalidad desd el inicio del desarrollo aseguroa que la API exihibira un comportamiento consistente apoyado en una metodologia orientada a testing.

Nota: Si el parametro de simulacion existe en el entorno de produccion, un mensaje de error deberia emerger.

## JSONP

JSONP es simple de explicar con un Ejemplo. Vamos a usar el siguiente [StackOverflow](http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top):

> Supongamos que estamos en el dominio abc.com, y queremos hacer un request al dominio xyz.com. Para hacer eso, hay que cruzar los limites de dominio, un no-no en la mayoria de los navegadores.

> Los unico que bypassea esta limitacion es `<script>` tags. Cuando usas un script tag, la limitacion por dominio es ignorada, pero bajo circunstancias normales, no se puede hacer nada con estos resultados, el script solo es evaluado.

> Agrega JSONP. Cuando haces un request a un servidor donde JSONP es enabled, se envia un parametro especial que informa al servidor sobre tu pagina. De esa manera el servidor puede empaquetar la respuesta en una manera que tu pagina puede administrarla.

> Por Ejemplo, digamos que el server espera un parametro llamado "callback" para activar JSONP. Entonces el request se veria de la siguiente manera:

>         http://www.xyz.com/sample.aspx?callback=mycallback

> Sin JSONP, esto devolveria un objeto basico de javascript:

>         { foo: 'bar' }

> Sin embargo, con JSONP, cuando el server recibe el parametro "callback", lo empaqueta de manera diferente y devuelve algo asi:

>         mycallback({ foo: 'bar' });

> Como pueden ver, ahora invoca el metodo especificado. Entonces en tu pagina podes definir la funcion callback:

>         mycallback = function(data){
>             alert(data.foo);
>         };

http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top
