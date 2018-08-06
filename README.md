Aero-Packs es la aplicacion donde Aero S.R.L ofrece todos sus servicios en un único servicio integral.
    
Este documento debe utilizarse como referencia para utilizar la API de AeroPacks. El mismo, esta en version beta. Pueden haber modificaciones o nueva información puede ser añadida.
    
Se necesitan credenciales de acceso para el uso de esta API. Todo acceso a endpoint esta protegido y necesita de un token de seguridad configurado mediante el CAS otorgando permisos para la API de la aplicacion.
        
## **INTRODUCCION**
    
El desarrollo actual de la aplicación back-end permite que cada salida tenga asociado un único destino, cupo aéreo y habitacion de hotel, pero la versatilidad del sistema podría permitir, a futuro, que esto pueda modificarse para la relacion múltiple de elementos de este tipo para cada salida puntual.
    
### **A tener en cuenta:**
     
Tener en cuenta que **un Paquete** se forma de **X Salidas**, utilizando muchos de los sistemas backend internos de la empresa, son la cantidad de cupos aéreos, hoteles y bases en las que se crea el paquete quienes determinan la cantidad de salidas que un paquete posee. 
    
Ejemplificando, **una salida esta comprendida por:** 

- Datos generales del conjunto de salidas (paquete), compuesto por:
  - Nombre del paquete
  
  - Categorías. Es una categorización a la que pertenece uno o un conjunto de paquetes, por ejemplo, 'Sol y Mar' o 'Aero Promociones'
  
  - Destino. El destino asociado al paquete, sirve de base para la busqueda de servicios, cupos aéreos y habitaciones.
  
  - Base. Puede ser SGL, DBL y/o TPL.
  
  - Fecha de Inicio
  
  - Fecha de Fin
  
  - Fecha de Validez de Inicio. (Opcional). Junto con el siguiente atributo formal fecha entre la que el paquete puede ser accedido para consulta y venta. 
  
  - Fecha de Validez de Fin. (Opcional)
  
  - Observaciones. (Opcional)

- Un Cupos Aéreo. (Obligatorio)

- Una Habitaciones de hotel. (Obligatorio)

- Servicios, pueden asociarse mas de un servicio por grupo de salidas, particularmente y en fin a una salida: 
  - Admisiones
  
  - Aéreo
  
  - Excursión
  
  - Genérico
  
  - Traslados. (Obligatorio al menos uno)
  
  - VISA

- Asistencia al viajero, de las cuales tambíen puede tener 1 o mas asociadas.
    
Hay algunos **atributos a tener en cuenta** sobre una salida:
    
Los primeros de ellos son una serie de claves generadas por medio de la codificación MD5, que sirven para identificar univocamente a los paquetes con características similares, a continuación detallamos las claves y la composición:
  - **Group_key:** ID del/los cupos aéreos + ID del/los aéreos de sistema + ID del/los destinos asociados + Numero de base.
  
  - **Unique_key:** Nombre + Fecha de Inicio + Fecha de Fin + ID del/los hoteles + ID del/las categorías de habitacion + Número de base + ID del/los destinos asociados.
  
  - **Different_base_group_key:** ID del/los cupos aéreos + ID del/los aéreos de sistema + ID del/los destinos asociados + ID del/los hoteles + ID del/las categorías de habitacion.
  
  - **Creation_group_key:** Nombre + ID del destino asociado.

El segundo grupo es un conjunto de atributos virtuales que se deben tener en consideración por el consumo de recursos al momento de calcularse:
  - **Price.** Recopilando la información de cada servicio que compone el paquete usando las API's correspondientes, se calcula el precio bruto de la salida.
  
  - **Price_with_markup.** Usando el markup se calcula el precio final de la salida con markup. El markup, puede obtenerse de distintas maneras: 
    - De la condición especial de la agencia que consume la API.
    
    - De la condicion base de la agencia que consume la API.
    
    - Del markup propio del paquete o la salida.
    
    - De la comision por defecto si la agencia no contiene condiciones, la misma es de 13%.
  
    Ambos atributos anteriores se devuelven en tres diferentes soportes de monedas: 
    - Peso Argentino (ARS).
    - Dolar Americano (USD).
    - Euro (EUR).
    - Libra (GBP).

Dos atributos más, se vuelven relevantes para el uso de la API pero son invisibles para el consumidor de la misma:
  - **Visibilidad.** Determina si la salida esta visible para la búsqueda por la API.
  
  - **Disponibilidad.** Se determina en base a las disponibilidades de los items de la salida, sobre esto podemos decir:
    - DISPONIBILIDAD EN HOTELES: es obligatorio que haya disponibilidad en este item, la misma depende de la fechas de inicio y fin de la salida ademas de:
      - Si no hay tarifas cargas - NO DISPONIBLE.
      
      - Si hay tarifas cargadas y hay un paro de ventas - NO DISPONIBLE.
      
      - Si hay tarifas cargadas y NO hay un paro de ventas - DISPONIBLE.
      
      - Si hay tarifas cargadas y hay un paro de ventas, pero el hotel tiene precompra cargada - DISPONIBLE.
    
    - DISPONIBILIDAD EN AEREOS: es obligatorio que haya disponibilidad en este item, para ello es necesario la fechas de inicio y fin de la salida.
    
    - DISPONIBILIDAD EN SERVICIOS: es obligatorio que haya disponibilidad en este item, para ello es necesario la fechas de inicio y fin de la salida.
    
    - DISPONIBILIDAD EN ASISTENCIAS: No es obligatira.
Teniendo en cuenta los aspectos mencionados, podremos hablar de las respuestas de los endpoints de la API.    

## **API**

### **Información de las respuestas**

- Todas las respuestas de esta API son en formato **JSON**. A continuación se presentan las claves raíz de cada respuesta de los endpoints: 
  - `status`: Indica si la respuesta fue exitosa o si termino con errores o notificaciones. Los posibles valores son: `ok`, `error`.
  
  - `notifications`: Es un String que representa a las notificaciones acordes al estado de la respjuesta, si la respuesta fue correcta nada se retorna aquí, pero si un error sucede, aquí aparecerá un detalle informativo de lo sucedido.
  
  - `info`: Indica la respuesta particular al endpoint que se esta haciendo la peticion, suele ser un Array con los elementos puntuales. Podemos diferenciar diferentes tipos de respuesta dependiendo del endpoint que se consume, en general:
    
Los endpoint suelen retornar objetos con caracteristicas similares dependiendo el tipo de endpoint requerido, siempre dentro del parámetro `info`. 

Particularmente dividiremos las respuestas en:
  
  - `búsquedas`:
    ```json
    [
      {
          ...datos generales de la salida...
          "categories": [],
          "services": [],
          "departures": [],
          "accomodation": [],
          "places": [],
          "images": [],
          "prices": {},
          "prices_with_markup": {},
          "related_packages": [],
          "base_related_packages": [],
          "other_related_packages": []
      }
    ]
    ```
  
  - `disponibilidad, garantizar y/o cancelar reserva`: Retorna `true`o `false`. 
  
  - `otras bases y opciones`: 
    ```json
    [
      {
        "aero_id": `integer`,
        "can_sell": `boolean`,
        "accomodation": [],
        "places": [],
        "images": [],
        "prices": {},
        "prices_with_markup": {}
        }
    ]
    ```
  
  - `reserva`: Falso en caso de fallo o, en caso de éxito: 
    ```json
    {
      "reserve_id": `integer`,
      "seller_email": `string`
    }
    ```

**NOTA:** La distribución de los pasajeros para los endpoints que requieran el parametro de entrada, debe ser igual a la distribución de la salida. Por ejemplo, si está buscando una salida para dos personas, debe respetar esto en el proceso de asignación de espacios. Esta última operación se explicará más adelante.        

  
### **1. Introducción**

Como se explico anteriormente, la API fue diseñada con el propósito de busqueda y reserva de salidas de paquetes por parte de diferentes agencias.

Hay algunos endpoints para obtener algunas salidas similares basandose en una con parametros parecidos, para eso nos basamos en los conceptos previamente definidos como `keys`dentro de cada salida, posteriormente explicaremos estos endpoints y su funcionamiento.

Para poder reservar una salida putual, o mas de una, es necesario generar una reserva que pase por el MO y sea exitosa, para ello, es necesario tener el identificador único de la salida a reservar, algunos datos de los pasajeros, asi como también, del vendedor. **Generar una reserva** implica reservar los **lugares** para los pasajeros y descontar del stock de la empresa los servicios, es por ello que es de vital importacia, gestionar bien los recursos y `dar de baja` una reserva o `garantizarla` de forma apropiada.

Un **lugar** es una habitación, un asiento o cualquier lugar genérico en el que se debe asignar un pasajero. Por ejemplo: en hoteles, los pasajeros deben estar asociados a las habitaciones, mientras que en los vuelos, un pasajero debe estar asociado a un asiento. Este proceso de asignación es necesario porque una reserva podría incluir muchos artículos y pasajeros, y la API debe contar de forma correcta con esta información para validar al momento de generarla. 

Una vez que los artículos y los pasajeros (además de su distribución respectiva) se usan para intentar generar la reserva, el proceso de reserva se completa con dos operaciones adicionales: ** verificar disponibilidad ** y ** garantizar **.

Por un lado, la verificación de disponibilidad es necesaria para garantizar la disponibilidad de los artículos, que es un proceso obligatorio para la mayoría de los proveedores. Por otro lado, y solo cuando la verificación ha tenido éxito, la operación de confirmación o garantía realiza la reserva real en el proveedor y en nuestros sistemas back-end.

Debe tener en cuenta que los artículos podrían tener ** tarifas adicionales diferentes **. El más común es la **tarifa de cancelación**, que es una cantidad que debe pagar si usted (o sus clientes) cancelan una reserva. Se expresa utilizando intervalos de tiempo porque en su mayoría dependen del tiempo entre la fecha de cancelación y la fecha de entrada de reserva. Estas tarifas están presentes en **hoteles por ejemplo**.

Todos los endpoints en las diferentes etapas del proceso de reserva tienen sus propias validaciones y administración de errores usando un formato de respuesta estándar ya visto anteriormete. Esta información se puede encontrar en la documentación de cada punto final de forma mas detallada.

### **2. Búsqueda de salidas**

El primer paso del proceso de reserva es la adquisición de la salida. 

La respuesta de búsqueda tiene el mismo formato estandarizado que el resto de los endpoints, cada uno personalizándolo para su propio propósito. Para cada operación de búsqueda, el campo `info` en la respuesta representa una colección (matriz) de salidas con un formato stardard. Este proceso también agrega información adicional al campo `status` para determinar si la respuesta fue exitosa o no. En caso de **no** serlo, el campo `notifications`incluye una breve descripción del error producido. 

Dentro de estos endpoints podemos encontrarnos diferenciados: 
  - `find`. Búsqueda de una salida puntual con el identificador único. 
  
  `GET /api/packages/:id/find`
  
  - `find_with_params`. Búsqueda que puede retornar varias salidas en base a parámetros de la búsqueda. 
  
  `POST /api/packages/find_with_params`
  
  - `index`. Resultado de todas las salidas disponibles al momento del llamado del endpoint. 
  
  `GET /api/packages`
  
Posteriormente se detallará cada endpoint con sus respectivos verbos y parámetros.
    
### **Estructura de la Salida**

La estructura de la Salida está estandarizada en todos los endpoints de búsqueda
  - Uno de los valores más importantes para el proceso de reserva es el `id`. Cada salida tiene un valor único de 'id' que se corresponde con el 'id' en el back-end.

  - El precio del artículo se puede obtener del campo `prices` y `prices_with_taxes`. Es un hash que contiene el valor expresado en las diferentes monedas admitidas. La diferencia entre estos campos es el valor agregado del impuesto con respecto del precio final en el segundo item mencionado.

  - Una salida esta compuesta de varios elementos que también tienen información adicional relacionada con su propia naturaleza (por ejemplo, `rooms`,` images`, `stars` y` regime` para hoteles, o `arrival_time`,` departure_time`, `arrival_airport` para vuelos). Esta información se encuentra dentro de cada elemento y está documentado mas adelante. También se incluye información interna de back-end que puede ser ignorada.

  - Todos los elementos también tienen su información de origen: su valor `service_kind` representa el tipo de elemento (` hotel`, `flight`,` transfer`, `land_product`,` assistance`, etc.).
  
  - Como se explicó anteriormente, los elementos estan comprendidos de diferente información de acuerdo a su tipo puntual, particularmente, veremos cada una en el formato del item de respuesta a continuación. 

**Formato de respuesta de la búsqueda**

```json
{
  "status": "ok",
  "notifications": [],
  "info": [
    [
      {...},
      {...},
    ]
  ]
}
```

**Formato de la salida y sus elementos** _(localizado dentro del campo ìnfo` de la respuesta anterior)_

```json
{
  "id": 1,
  "name": "Solicitud inicial ",
  "description": "Paquete Masivo N°1 - Cupo LA 19Jul18 8 nts - RIU Cancún - Doble Estándar Vista Mar (DBMB)",
  "available": false,
  "observations": "",
  "start_date": "2018-07-19",
  "end_date": "2018-07-27",
  "base": "SGL",
  "categories": [
    {
      "name": "Sol y Mar"
    }
  ],
  "services": [
    {
      "detail": "Asistencia al viajero Master ",
      "service_kind": "travel_assistance"
    },
    {...}
  ],
  "departures": [
    {
      "flight_number": "LA1450",
      "date": "2018-07-19",
      "departure_airport": "EZE",
      "arrival_airport": "LIM",
      "departure_time": "0355",
      "arrival_time": "0655",
      "date_arrival": "2018-07-19",
      "direction": 1
    },
    {...}
  ],
  "accomodation": [
      {
          "name": "RIU Cancún",
          "category_name": "Doble Estándar Vista Mar (DBMB)",
          "service_kind": "hotel",
          "room_type": "single",
          "stars": 5,
          "regime": "Todo incluido",
          "check_in": "2018-07-19",
          "check_out": "2018-07-27",
          "nights": 8,
          "city_iata": "CUN"
      }
  ],
  "places": [
      {
          "name": "México, Cancún",
          "country": "México",
          "iata_code": "CUN"
      }
  ],
  "images": [],
  "prices": {
      "ars_value": 90369.46,
      "usd_value": 3250.7,
      "eur_value": 2712.17,
      "gbp_value": 2338.63
  },
  "prices_with_markup": {
      "ars_value": 107054.46,
      "usd_value": 3850.88,
      "eur_value": 3212.92,
      "gbp_value": 2770.42
  },
  "related_packages": [
      2,
      5
  ],
  "base_related_packages": [
      2
  ],
  "other_related_packages": [
      5
  ]
} 
```

Los campos `related_packages`, `base_related_packages`, y `other_related_packages`  nos retornan un arreglo de identificadores únicos de las salidas que se relacionan con la salida encontrada en la respuesta basandose en la clave que relaciona las salidas de acuerdo a:
  - `related_packages` -> Creation_group_key 
  
  - `base_related_packages` -> Different_base_group_key
  
  - `other_related_packages` -> Group_key

#### **A. Busquedas de salidas relacionadas**

Como se mencioó anteriormente de acuerdo a una clave generada se pueden encontrar salidas relacionadas en base a un concepto puntual, para ello se disponen de dos endpoints distintos, los cuales son:
  - `other_bases`. Se centra en encontrar las salidas relacionadas con la salida puntual determinada por el identificador único en la llamada basándose en las bases del paquete creado. 
  
  `GET /api/packages/:id/other_bases`
  
  - `other_options`. Se centra en encontrar las salidas relacionadas con la salida puntual determinada por el identificador único en la llamada basándose en los lugares y cupos aéreos del paquete creado. 
  
  `GET /api/packages/:id/other_options`

**Formato de respuesta de la búsqueda**

```json
{
  "status": "ok",
  "notifications": [],
  "info": []
}
```

**Formato de la salida y sus elementos** _(localizado dentro del campo ìnfo` de la respuesta anterior)_

```json
{
  "aero_id": 24,
  "can_sell": true,
  "accomodation": [
      {
        "aero_id": 317,
        "aero_category_id": 789,
        "name": "RIU Lupita",
        "category_name": "Standard (DBSB)",
        "service_kind": "hotel",
        "room_type": "double",
        "stars": 5,
        "regime": "Todo incluido",
        "check_in": "2018-01-05",
        "check_out": "2018-01-12",
        "nights": 7
      }
  ],
  "images": [],
  "prices": {
    "ars_value": 82460.8,
    "usd_value": 4672,
    "eur_value": 3849.71,
    "gbp_value": 3566.41
  },
  "prices_with_markup": {
    "ars_value": 95288.82,
    "usd_value": 5398.8,
    "eur_value": 4448.59,
    "gbp_value": 4121.22
  }
},
{...}
```   

### **3. Gestión de reservas**
Como se explicó en la introducción, una reserva se forma por una colección de artículos que se pueden reservar. El proceso de reserva implica las siguientes operaciones de manera implícita para la API:
    - Crear reserva.
    
    - Agregar elementos a la reserva. (Implícito)
    
    - Agregar pasajeros a la reserva. (Implícito)
    
    - Asignar pasajeros a los puntos de reserva. (Implícito)
    
    - Verificar reserva. (Implícito)
    
    - Confirmar reserva.

  Análogamente, se puede pedir una cancelación de una reserva cuando se considere necesario, de esta manera se desocupan los **lugares** asignados previamente para poder ser reutilizados, para ello, disponemos de:
    - Cancelar reserva.
  
#### **A. Crear una reserva**

Este proceso interno crea una reserva con un `id` único, que se devuelve como parte de los datos de la reserva en el campo `reserve_id` en la clave `info`. Este valor es necesario para administrar la reserva como recurso en las siguientes etapas del proceso de reserva.

Además de eso, los parámetros requeridos de la reserva también contienen información relacionada con los pasajeros que se han de agregae a la misma. Cabe destacar que la cantidad de elementos del arreglo de pasajeros debe corresponderse con la cantidad de adultos que se envían como paámetro. A continuación vemos un ejemplo de parámetros requeridos para el uso del endpoint y de su respuesta.

`POST /api/packages/:id/reserve`

**Formato de request**

```json
{
  "booking_params": {
    "adults_number": `integer`,
    "reserve_data" : {
      "seller_email": `string`,
      "agency_id": `integer`
    },
    "passengers": [
      {
        "first_name": `string`,
        "last_name": `string`,
        "document_type": "DNI",
        "document_number": `string number`,
        "birthdate": `string date`
      },
      {...}
    ]
  }
}
``` 

**Formato de respuesta de la reserva**

```json
{
  "status": "ok",
  "notifications": "Se ha enviado el mail a jonatan.sivori@aerolaplata.com.ar",
  "info": {
    "reserve_id": 35092,
    "seller_email": "jonatan.sivori@aerolaplata.com.ar"
  }
}

O en caso de no poder gestionarse la reserva

{
  "status": "error",
  "notifications": "El paquete seleccionado no puede reservarse, no se encuentra disponible",
  "info": []
}
``` 

#### **B. Confirmar reserva**

Este endpoint envía un mail al vendedor u operativo asociado a la reserva para solicitar la confirmacion de la misma asociada a un paquete. El identificador unico de la reserva debe corresponderse con el identificador unico de reserva dentro de nuestro back-end de gestión de reservas.

Como se explicó anteriormente, la verificación es necesaria para garantizar la disponibilidad de los elementos, que es un proceso obligatorio para la mayoría de los proveedores. Debe verificar la reserva antes de su proceso de confirmación utilizando el endpoint
  
`GET api/packages/reserve_to_guarantee?reserve_id=:id_reserve`.

Una vez que el llamado finaliza, la verificación devuelve diferentes valores de `status` de acuerdo con el resultado:

  - `ok`: todos los elementos están disponibles y se resevaron.
  
  - `error`: algun elemento no está disponibles o se produjo un error.

`GET api/packages/reserve_to_guarantee?reserve_id=:id_reserve`

**Formato de respuesta de la reserva**

```json
{
  "status": "ok",
  "notifications": "Se ha enviado el mail a juliana.soto@aerolaplata.com.ar",
  "info": true
}
``` 

El campo `notifications` nos retorna información sobre la culminación del endpoint y se corresponde con el estado en el que la acción terminó.

#### **C. Cancelar reserva**
Este endpoint envía un mail al vendedor u operativo asociado a la reserva para solicitar la cancelación de la misma asociada a un paquete. El identificador unico de la reserva debe corresponderse con el identificador unico de reserva dentro de nuestro back-end de gestión de reservas.

Esta acción se lleva a cabo utilizando el endpoint `GET api/packages/reserve_to_cancel?reserve_id=:id_reserve`.

Una vez que el llamado finaliza, la verificación devuelve diferentes valores de `status` de acuerdo con el resultado:
  
  - `ok`: se envió un mail para cancelar la reserva al operativo.
  
  - `error`: se produjo un error.
  
`GET api/packages/reserve_to_cancel?reserve_id=:id_reserve`

**Formato de respuesta de la reserva**

```json
{
  "status": "ok",
  "notifications": "Se ha enviado el mail a juliana.soto@aerolaplata.com.ar",
  "info": true
}
``` 

El campo `notifications` nos retorna información sobre la culminación del endpoint y se corresponde con el estado en el que la acción terminó.
