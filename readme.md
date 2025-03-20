
# Diferencias entre v1 y v2

Este README explica las diferencias clave entre la versión 1 (v1) y la versión 2 (v2) del código, para que quede claro cuál es la finalidad de cada versión y se pueda documentar adecuadamente en GitHub.

---

## 1. Criterios de Filtrado

### v1 – Notificaciones de Stock
- **Condición de Filtrado:**  
  La versión 1 filtra los elementos donde se cumple que:
  ```js
  item.notif_info && item.notif_info.tipo_objeto === "stock";
  ```
  Esto significa que se retienen únicamente las notificaciones relacionadas con cambios en el stock.

- **Campo de Salida:**  
  Los elementos filtrados se almacenan en la propiedad `filteredItems` del objeto JSON de salida.

### v2 – Notificaciones de Órdenes
- **Condición de Filtrado:**  
  La versión 2 filtra las notificaciones para mantener solo aquellas que corresponden a órdenes (nuevas o modificadas). Se utilizan dos condiciones:
  1. `notif_info.tipo_objeto` debe ser igual a `"comprobantes"`.
  2. El objeto `data` debe contener un identificador de orden (`cpr_id`) definido.
  
  Esto se expresa en el siguiente fragmento de código:
  ```js
  item.notif_info &&
  item.notif_info.tipo_objeto === "comprobantes" &&
  item.data &&
  item.data.cpr_id !== undefined;
  ```
  De esta forma, se descartan las notificaciones relacionadas con cambios en el stock, que no son de interés.

- **Campo de Salida:**  
  Los resultados filtrados se asignan a la propiedad `orders` del objeto JSON de salida.

---

## 2. Nomenclatura y Salida

- **v1:**  
  - Los resultados filtrados se guardan en `$input.item.json.filteredItems`.
  - Está pensada para casos en los que el interés es monitorizar los cambios en el stock.

- **v2:**  
  - Los resultados filtrados se guardan en `$input.item.json.orders`.
  - Está diseñada para enfocarse únicamente en las notificaciones de órdenes, es decir, aquellas que indican nuevas órdenes o actualizaciones en órdenes existentes (identificadas por la presencia de `cpr_id`).

---

## 3. Casos de Uso y Propósito

- **v1 – Notificaciones de Stock:**  
  Usa esta versión si tu interés radica en procesar y visualizar notificaciones relacionadas con el stock (por ejemplo, aumento o disminución de inventario).

- **v2 – Notificaciones de Órdenes:**  
  Usa esta versión si únicamente te interesan las notificaciones de órdenes, es decir, las notificaciones que indican nuevas órdenes o modificaciones en órdenes existentes. En esta versión se retienen sólo los elementos en los que:
  - `notif_info.tipo_objeto` es `"comprobantes"`.
  - Existe un identificador de orden (`cpr_id`) en el objeto `data`.

---

## Ejemplos de Código

### Código de v1 (Notificaciones de Stock)
```js
// Obtener el payload del item de entrada.
const rawPayload = $input.item.json.payload;

// Asegurarse de tener un arreglo.
// Si rawPayload es una cadena, se intenta parsearla;
// si ya es un arreglo, se utiliza directamente.
let itemsArray;
if (typeof rawPayload === 'string') {
  try {
    itemsArray = JSON.parse(rawPayload);
  } catch (e) {
    throw new Error("Error al parsear la cadena del payload: " + e);
  }
} else if (Array.isArray(rawPayload)) {
  itemsArray = rawPayload;
} else {
  itemsArray = [rawPayload];
}

// Filtrar las notificaciones de stock.
const filteredItems = itemsArray.filter(item => {
  return item.notif_info && item.notif_info.tipo_objeto === "stock";
});

// Agregar el arreglo filtrado en una nueva propiedad.
$input.item.json.filteredItems = filteredItems;

// Devolver el item actualizado.
return $input.item;
```

### Código de v2 (Notificaciones de Órdenes)
```js
// Obtener el payload del JSON de entrada (puede ser una cadena o un arreglo).
const rawPayload = $input.item.json.payload;

// Parsear rawPayload si es una cadena; si ya es un arreglo (o un objeto), usarlo.
let parsedArray;
if (typeof rawPayload === 'string') {
  try {
    parsedArray = JSON.parse(rawPayload);
  } catch (e) {
    throw new Error("Error al parsear la cadena del payload: " + e.message);
  }
} else if (Array.isArray(rawPayload)) {
  parsedArray = rawPayload;
} else {
  parsedArray = [rawPayload];
}

// Filtrar las notificaciones de órdenes.
// Se retienen los elementos donde:
//  • notif_info.tipo_objeto === "comprobantes"
//  • data contiene un cpr_id definido (identificador de la orden).
const orders = parsedArray.filter(item => {
  return item.notif_info && 
         item.notif_info.tipo_objeto === "comprobantes" &&
         item.data && 
         item.data.cpr_id !== undefined;
});

// Agregar el arreglo filtrado (aunque esté vacío) en una nueva propiedad "orders".
$input.item.json.orders = orders;

// Devolver el item actualizado.
return $input.item;
```

---

## Conclusión

- **v1** se utiliza para procesar y mostrar notificaciones de cambios en el stock, almacenando los resultados en `filteredItems`.
- **v2** se utiliza para capturar únicamente notificaciones relacionadas con órdenes (nuevas o actualizadas), filtrando los elementos que tienen `notif_info.tipo_objeto` igual a `"comprobantes"` y que poseen un identificador de orden (`cpr_id`). Los resultados se guardan en `orders`.

Esta diferenciación te permitirá documentar claramente en GitHub cuál versión usar en función de la información que necesites procesar.

```
