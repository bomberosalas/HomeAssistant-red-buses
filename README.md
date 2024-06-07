# HomeAssistant-red-buses
Plantilla para home assistant que permite saber los próximos buses que llegan a un determinado paradero

## Descripción


*Este código, corresponde a una simple implementación de una plantilla **REST** que consume una api de [red movilidad](https://www.red.cl/), que es el estamento en Chile que entrega información acerca de la red de transporte metropolitano en Santiago*

<h1 align="center">Plantilla Red Movilidad</h1>
<p align="center"><img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTwG-4Jk9MEcyjPoLYpUvbmy7PQc9ynD66zY7ZbOB9GdLVoNJbbxtn32WclNqmQ7f9bWjA&usqp=CAU"/></p> 

## Implementación
---
Si ya está familiarizado con la edición de sensores en Home Assistant, entiende que todas las personalizaciones de sensores e integraciones se deben realizar a nivel de código, para lo cual debe editar el archivo `configuration.yaml`, para lo cual debe tener habilitado un editor de texto o visual studio code en el mismo Home Assistant.

### Recurso
Para la buena implementación de este sensor, debe definir cual es la dirección o enlace al cual debe apuntar su petición, en este caso a `https://www.red.cl/predictor/prediccion?t=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3MTYzMjc2NjgzMTB9.e5RR9mTdrcEKFfILzV0fYg4Pf3_QWbb2nOCLgKrtY88&codsimt=PA23&codser=`, donde el valor **PA23**, `codsimt=PA23`
debe ser reemplazado por el paradero que desee monitorear.

La respuesta a esa petición será devuelta en formato `JSON`, del cual se pueden extraer muchos otros datos, pero en esta implementación nos enfocamos en el número del recorrido y del tiempo de llegada de los próximos buses.

---
### Respuesta JSON para PA436
---
```json
// 20240607012743
// https://www.red.cl/predictor/prediccion?t=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3MTYzMjc2NjgzMTB9.e5RR9mTdrcEKFfILzV0fYg4Pf3_QWbb2nOCLgKrtY88&codsimt=PA436&codser=

{
  "fechaprediccion": "2024-06-07",
  "horaprediccion": "01:27",
  "nomett": "Avenida Matta / esq. Nataniel Cox",
  "paradero": "PA436",
  "respuestaParadero": "",
  "servicios": {
    "item": [
      {
        "codigorespuesta": "00",
        "distanciabus1": "2330",
        "distanciabus2": "12499",
        "horaprediccionbus1": "Menos de 8 min.",
        "horaprediccionbus2": "Entre 22 Y 30 min. ",
        "ppubus1": "GCBD-74",
        "ppubus2": "PFXB-62",
        "respuestaServicio": "",
        "servicio": "506",
        "color": "#0093B3",
        "destino": "Peñalolén",
        "sentido": "1",
        "itinerario": true,
        "codigo": "U5"
      },
      {
        "codigorespuesta": "11",
        "distanciabus1": null,
        "distanciabus2": null,
        "horaprediccionbus1": null,
        "horaprediccionbus2": null,
        "ppubus1": null,
        "ppubus2": null,
        "respuestaServicio": "Fuera de horario de operacion para este paradero",
        "servicio": "506V",
        "color": "#0093B3",
        "destino": "Peñalolén",
        "sentido": "1",
        "itinerario": false,
        "codigo": "U5"
      },
      {
        "codigorespuesta": "11",
        "distanciabus1": null,
        "distanciabus2": null,
        "horaprediccionbus1": null,
        "horaprediccionbus2": null,
        "ppubus1": null,
        "ppubus2": null,
        "respuestaServicio": "Fuera de horario de operacion para este paradero",
        "servicio": "507",
        "color": "#0093B3",
        "destino": "Av. Grecia",
        "sentido": "1",
        "itinerario": false,
        "codigo": "U5"
      },
      {
        "codigorespuesta": "11",
        "distanciabus1": null,
        "distanciabus2": null,
        "horaprediccionbus1": null,
        "horaprediccionbus2": null,
        "ppubus1": null,
        "ppubus2": null,
        "respuestaServicio": "Fuera de horario de operacion para este paradero",
        "servicio": "507C",
        "color": "#0093B3",
        "destino": "Peñalolén",
        "sentido": "1",
        "itinerario": false,
        "codigo": "U5"
      },
      {
        "codigorespuesta": "11",
        "distanciabus1": null,
        "distanciabus2": null,
        "horaprediccionbus1": null,
        "horaprediccionbus2": null,
        "ppubus1": null,
        "ppubus2": null,
        "respuestaServicio": "Fuera de horario de operacion para este paradero",
        "servicio": "510",
        "color": "#0093B3",
        "destino": "Río Claro",
        "sentido": "1",
        "itinerario": false,
        "codigo": "U5"
      }
    ]
  },
  "urlLinkPublicidad": null,
  "urlPublicidad": "http://mkt.smsbus.cl/mkt/img/app_transantiago_320_118b.png",
  "x": "-33.4600341",
  "y": "-70.6518149"
}

```
---
### Plantilla REST

El texto que debemos agregar al archivo `configuration.yaml` es el siguiente :

```yaml
rest:
  - resource: https://www.red.cl/predictor/prediccion?t=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3MTYzMjc2NjgzMTB9.e5RR9mTdrcEKFfILzV0fYg4Pf3_QWbb2nOCLgKrtY88&codsimt=PA23&codser=
    scan_interval: 30
    method: GET
    headers:
      Content-Type: "application/json"
    sensor:
      - name: bus0
        value_template: "{{ value_json.servicios.item[0].servicio}}"
      - name: llegada_bus01
        value_template: "{{value_json.servicios.item[0].horaprediccionbus1|replace('None', 'Fuera de Servicio')}}"
      - name: llegada_bus02
        value_template: "{{value_json.servicios.item[0].horaprediccionbus2|replace('None', 'Fuera de Servicio')}}"
      - name: bus1
        value_template: "{{ value_json.servicios.item[1].servicio}}"
      - name: llegada_bus11
        value_template: "{{value_json.servicios.item[1].horaprediccionbus1|replace('None', 'Fuera de Servicio')}}"
      - name: llegada_bus12
        value_template: "{{value_json.servicios.item[1].horaprediccionbus2|replace('None', 'Fuera de Servicio')}}"
      - name: bus2
        value_template: "{{ value_json.servicios.item[2].servicio}}"
      - name: llegada_bus21
        value_template: "{{value_json.servicios.item[2].horaprediccionbus1|replace('None', 'Fuera de Servicio')}}"
      - name: llegada_bus22
        value_template: "{{value_json.servicios.item[2].horaprediccionbus2|replace('None', 'Fuera de Servicio')}}"
```
## Vista al agregar el sensor al dashboard
---
Luego agregamos las entidades generadas a una tarjeta en nuestro dashboard, para que se vea de la siguiente forma :


