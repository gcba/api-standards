# Buenos Aires API Standards

* [Guidelines](#guidelines)
* [Pragmatic REST](#pragmatic-rest)
* [RESTful URLs](#restful-urls)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [Error handling](#error-handling)
* [Versiones](#versiones)
* [Record limits](#record-limits)
* [Request & Response Examples](#request-response-examples)
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
    * http://ejemplo.gob.ar/api/v1/informe.json
    * http://ejemplo.gob.ar/api/v1/informe.xml

## RESTful URLs

### Guidelines generales para RESTful URLs
* Una URL identifica un recurso.
* URLs deben incluir nombres, no verbos.
* Utilice nombres en plural sólo por coherencia (sin nombres en singular).
* Utilice HTTP verbs (GET, POST, PUT, DELETE) para operar en las colecciones y elementos..
* Usted no necesita ir más allá de recurso/identificador/recurso.
* Ponga el número de versión en la base de la URL, por ejemplo http://ejemplo.gob.ar/v1/path/to/recurso.
* URL v. header:
    * Si cambia la logica para administrar la respuesta, agregarlo en la URL.
    * Si no modifica la logica para cada respuesta, como OAuth info, agregarlo en el Header.
* Especificar campos opcionales en una lista separada por comas.
* Los Formatos deben estar en la forma de api/v2/recurso/{id}.json

### Ejemplos de un Correcto uso de las URL
* Lista de Informes:
    * http://www.ejemplo.gob.ar/api/v1/informes.json
* Filtrando una query:
    * http://www.ejemplo.gob.ar/api/v1/informes.json?year=2011&sort=desc
    * http://www.ejemplo.gob.ar/api/v1/informes.json?topic=economy&year=2011
* Un unico informe en formato JSON:
    * http://www.ejemplo.gob.ar/api/v1/informes/1234.json
* Todos los articulos en (o pertenecen a) este informe:
    * http://www.ejemplo.gob.ar/api/v1/informes/1234/articles.json
* Todos los articulos de este informe en formato XML:
    * GET http://ejemplo.gob.ar/api/v1/informes/1234/articles.xml
* Especificar campos opcionales en una lista separada por comas:
    * http://www.ejemplo.gob.ar/api/v1/informes/1234.json?fields=titulo,subtitulo,fecha
* Agregar un articulo a un informe:
    * POST http://ejemplo.gob.ar/api/v1/informes/1234/articles

### Usos NO CORRECTOS de URL
* Nombres en singular:
    * http://www.ejemplo.gob.ar/informe
    * http://www.ejemplo.gob.ar/informe/1234
    * http://www.ejemplo.gob.ar/publicador/informe/1234
* Verbo en URL:
    * http://www.ejemplo.gob.ar/informe/1234/crear
* Filtro fuera de la query de consulta
    * http://www.ejemplo.gob.ar/informes/2011/desc

## HTTP Verbs

| HTTP METHOD | POST            | GET       | PUT         | DELETE |
| ----------- | --------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE          | READ      | UPfecha      | DELETE |
| /perros       | Create nuevo perro | List perros | Bulk upfecha | Delete all perros |
| /perros/1234  | Error           | Show perro   | If exists, update perro; If not, error | Delete perro |

(Example from Web API Design, by Brian Mulloy, Apigee.)


## Responses

* No values in keys
* No internal-specific names (e.g. "node" and "taxonomy term")
* Metadata should only contain direct properties of the response set, not properties of the members of the response set

### Good examples

No values in keys:

    "tags": [
      {"id": "125", "name": "Environment"},
      {"id": "834", "name": "Water Quality"}
    ],


### Bad examples

Values in keys:

    "tags": [
      {"125": "Environment"},
      {"834": "Water Quality"}
    ],


## Error handling

Error responses should include a common HTTP status code, message for the developer, message for the end-user (when appropriate), internal error code (corresponding to some specific internally determined error number), links where developers can find more info. For example:

    {
      "status" : "400",
      "developerMessage" : "Verbose, plain language description of the problem. Provide developers
       suggestions about how to solve their problems here",
      "userMessage" : "This is a message that can be passed along to end-users, if needed.",
      "errorCode" : "444444",
      "more info" : "http://www.ejemplo.gob.ar/developer/path/to/help/for/444444,
       http://drupal.org/node/444444",
    }

Use three simple, common response codes indicating (1) success, (2) failure due to client-side problem, (3) failure due to server-side problem:
* 200 - OK
* 400 - Bad Request
* 500 - Internal Server Error


## Versions

* Never release an API without a version number.
* Versions should be integers, not decimal numbers, prefixed with ‘v’. For example:
    * Good: v1, v2, v3
    * Bad: v-1.1, v1.2, 1.3
* Maintain APIs at least one version back.


## Record limits

* If no limit is specified, return results with a default limit.
* To get records 50 through 75 do this:
    * http://ejemplo.gob.ar/informes?limit=25&offset=50
    * offset=50 means, ‘begin with record number fifty’
    * limit=25 means, ‘return 25 records’

Information about record limits should also be included in the Example resonse. Example:

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

## Request & Response Examples

### API Resources

  - [GET /informes](#get-informes)
  - [GET /informes/[id]](#get-informesid)
  - [POST /informes/[id]/articles](#post-informesidarticles)

### GET /informes

Example: http://ejemplo.gob.ar/api/v1/informes.json

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
                "titulo": "Public Water Systems",
                "tags": [
                    {"id": "125", "name": "Environment"},
                    {"id": "834", "name": "Water Quality"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "informe",
                "titulo": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Elementary"},
                    {"id": "834", "name": "Charter Schools"}
                ],
                "created": "126251302"
            }
            {
                "id": 2351,
                "type": "informe",
                "titulo": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Pre-school"},
                ],
                "created": "126251302"
            }
        ]
    }

### GET /informes/[id]

Example: http://ejemplo.gob.ar/api/v1/informes/[id].json

    {
        "id": "1234",
        "type": "informe",
        "titulo": "Public Water Systems",
        "tags": [
            {"id": "125", "name": "Environment"},
            {"id": "834", "name": "Water Quality"}
        ],
        "created": "1231621302"
    }



### POST /informes/[id]/articles

Example: Create – POST  http://ejemplo.gob.ar/api/v1/informes/[id]/articles

    {
        "titulo": "Raising Revenue",
        "author_first_name": "Jane",
        "author_last_name": "Smith",
        "author_email": "jane.smith@ejemplo.gob.ar",
        "year": "2012"
        "month": "August"
        "day": "18"
        "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam eget ante ut augue scelerisque ornare. Aliquam tempus rhoncus quam vel luctus. Sed scelerisque fermentum fringilla. Suspendisse tincidunt nisl a metus feugiat vitae vestibulum enim vulputate. Quisque vehicula dictum elit, vitae cursus libero auctor sed. Vestibulum fermentum elementum nunc. Proin aliquam erat in turpis vehicula sit amet tristique lorem blandit. Nam augue est, bibendum et ultrices non, interdum in est. Quisque gravida orci lobortis... "

    }


## Mock Responses
It is suggested that each resource accept a 'mock' parameter on the testing server. Passing this parameter should return a mock data response (bypassing the backend).

Implementing this feature early in development ensures that the API will exhibit consistent behavior, supporting a test driven development methodology.

Note: If the mock parameter is included in a request to the production environment, an error should be raised.


## JSONP

JSONP is easiest explained with an example. Here's a one from [StackOverflow](http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top):

> Say you're on domain abc.com, and you want to make a request to domain xyz.com. To do so, you need to cross domain boundaries, a no-no in most of browserland.

> The one item that bypasses this limitation is `<script>` tags. When you use a script tag, the domain limitation is ignored, but under normal circumstances, you can't really DO anything with the results, the script just gets evaluated.

> Enter JSONP. When you make your request to a server that is JSONP enabled, you pass a special parameter that tells the server a little bit about your page. That way, the server is able to nicely wrap up its response in a way that your page can handle.

> For example, say the server expects a parameter called "callback" to enable its JSONP capabilities. Then your request would look like:

>         http://www.xyz.com/sample.aspx?callback=mycallback

> Without JSONP, this might return some basic javascript object, like so:

>         { foo: 'bar' }

> However, with JSONP, when the server receives the "callback" parameter, it wraps up the result a little differently, returning something like this:

>         mycallback({ foo: 'bar' });

> As you can see, it will now invoke the method you specified. So, in your page, you define the callback function:

>         mycallback = function(data){
>             alert(data.foo);
>         };

http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top
