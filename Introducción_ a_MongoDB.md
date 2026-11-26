# Introducción a MongoDB

## 1. Entorno y conexión

Usaremos Docker Compose. Crea un archivo `compose.yml`:

```yaml
services:
  mongo:
    image: mongo:7
    container_name: mongo
    ports:
      - '27017:27017'
    volumes:
      - mongo_data:/data/db
volumes:
  mongo_data:
```

| Comando | Función |
| --- | --- |
| `docker compose up -d` | Levanta MongoDB en segundo plano. |
| `docker compose ps` | Comprueba que el contenedor está en marcha. |
| `docker compose logs -f mongo` | Muestra los registros del servicio. |
| `docker compose exec mongo mongosh` | Abre la shell dentro del contenedor. |
| `docker compose stop` / `start` | Detiene / arranca los servicios existentes. |
| `docker compose down` | Elimina contenedores y redes, conservando los volúmenes. |
| `docker compose down -v` | Elimina también los volúmenes: **borra todos los datos**. |

**Cadena de conexión** (para Compass y para Java):

```
mongodb://localhost:27017
```

En Compass, pega esa cadena en *New connection*. Si trabajas con replica set (apartado 15), añade `/?directConnection=true`.

## 2. Datos de ejemplo

Ejecuta este bloque una vez en `mongosh`. Si lo repites, ejecuta antes `db.dropDatabase()` para no duplicar datos.

```jsx
use tienda

db.productos.insertMany([
  { codigo: 'P001', nombre: 'Ratón', categoria: 'informatica', precio: 20, coste: 12, stock: 35, etiquetas: ['gaming', 'oficina'] },
  { codigo: 'P002', nombre: 'Teclado mecánico', categoria: 'informatica', precio: 85, coste: 50, stock: 12, etiquetas: ['gaming'], descuento: 10 },
  { codigo: 'P003', nombre: 'Monitor 27 pulgadas', categoria: 'informatica', precio: 230, coste: 170, stock: 0, etiquetas: ['oficina', 'diseño'] },
  { codigo: 'P004', nombre: 'Smartphone X', categoria: 'movil', precio: 499, coste: 380, stock: 8, etiquetas: ['oferta'], descuento: 15 },
  { codigo: 'P005', nombre: 'Funda móvil', categoria: 'movil', precio: 12, coste: 3, stock: 120, etiquetas: ['accesorio'] },
  { codigo: 'P006', nombre: 'Auriculares', categoria: 'audio', precio: 60, coste: 35, stock: 25, etiquetas: ['gaming', 'audio'] },
  { codigo: 'P007', nombre: 'Altavoz Bluetooth', categoria: 'audio', precio: 45, coste: 28, stock: 0, etiquetas: ['oferta', 'audio'] },
  { codigo: 'P008', nombre: 'Portátil 15 pulgadas', categoria: 'informatica', precio: 750, coste: 600, stock: 5, etiquetas: ['oficina'], descuento: 5 }
])
	db.clientes.insertMany([
	  { _id: 1, nombre: 'Ana', ciudad: 'Madrid' },
	  { _id: 2, nombre: 'Luis', ciudad: 'Valencia' }
	])

db.pedidos.insertMany([
  { _id: 101, clienteId: 1, fecha: ISODate('2026-10-05'), lineas: [{ codigo: 'P001', cantidad: 2 }, { codigo: 'P006', cantidad: 1 }] },
  { _id: 102, clienteId: 2, fecha: ISODate('2026-10-08'), lineas: [{ codigo: 'P004', cantidad: 1 }] },
  { _id: 103, clienteId: 1, fecha: ISODate('2026-11-02'), lineas: [{ codigo: 'P002', cantidad: 1 }, { codigo: 'P005', cantidad: 3 }] }
])
```

<aside>
📌

Fíjate en que no todos los productos tienen el campo `descuento`: en MongoDB los documentos de una misma colección **no tienen por qué tener la misma estructura**.

</aside>

## 3. Conceptos básicos

| SQL | MongoDB |
| --- | --- |
| Base de datos | Base de datos |
| Tabla | Colección |
| Fila | Documento (JSON/BSON) |
| Columna | Campo |
| `JOIN` | `$lookup` o documentos embebidos |
| Clave primaria | `_id` (siempre existe) |

MongoDB guarda los documentos en **BSON**, un JSON binario con más tipos de datos.

| Tipo | Ejemplo en `mongosh` | Notas |
| --- | --- | --- |
| String | `'Ratón'` | Texto UTF-8. |
| Número | `20`, `9.99` | Por defecto, `mongosh` guarda los números como `double`. Para enteros: `NumberInt(20)`. |
| Booleano | `true` |  |
| Fecha | `ISODate('2026-10-05')` | Se guarda en UTC. |
| ObjectId | `ObjectId()` | Identificador de 12 bytes que se genera solo si no pones `_id`. |
| Array | `['a', 'b']` |  |
| Documento anidado | `{ calle: 'Mayor', num: 3 }` |  |
| Null | `null` |  |

## 4. Bases de datos y colecciones

| Comando | Función |
| --- | --- |
| `show dbs` | Lista las bases de datos (solo las que tienen datos). |
| `use tienda` | Selecciona una base de datos; se crea al guardar datos. |
| `db` | Muestra la base de datos actual. |
| `db.dropDatabase()` | Elimina la base de datos actual. |
| `show collections` | Lista las colecciones. |
| `db.createCollection('productos')` | Crea una colección explícitamente. |
| `db.productos.renameCollection('articulos')` | Renombra una colección. |
| `db.productos.drop()` | Elimina la colección y sus documentos. |

<aside>
📌

Las colecciones se crean solas al insertar el primer documento.

</aside>

## 5. Insertar documentos (CREATE)

| Comando | Función |
| --- | --- |
| `insertOne({...})` | Inserta un documento. |
| `insertMany([{...}, {...}])` | Inserta varios documentos. |
| `bulkWrite([...])` | Ejecuta varias operaciones de escritura en lote. |

```jsx
db.productos.insertOne({ codigo: 'P009', nombre: 'Webcam', categoria: 'informatica', precio: 40, stock: 10 })
```

<aside>
📌

Si no indicas `_id`, MongoDB genera un `ObjectId`. 

Si lo indicas y ya existe, obtendrás un error de clave duplicada.

</aside>

## 6. Consultar (READ)

| Comando | Función |
| --- | --- |
| `db.productos.find()` | Todos los documentos. |
| `db.productos.findOne({ codigo: 'P001' })` | Un solo documento. |
| `db.productos.find({ categoria: 'audio' })` | Filtro por igualdad. |
| `db.productos.find({}, { nombre: 1, _id: 0 })` | Proyección: solo el nombre. |
| `db.productos.countDocuments({ stock: 0 })` | Cuenta los que cumplen la condición. |
| `db.productos.distinct('categoria')` | Valores distintos de un campo. |

En una proyección, `1` incluye el campo y `0` lo excluye. No se pueden mezclar, salvo con `_id`.

### Operadores de comparación

| Operador | Significado | Ejemplo |
| --- | --- | --- |
| `$eq` / `$ne` | Igual / distinto | `{ precio: { $ne: 20 } }` |
| `$gt` / `$gte` | Mayor / mayor o igual | `{ precio: { $gt: 100 } }` |
| `$lt` / `$lte` | Menor / menor o igual | `{ precio: { $lte: 50 } }` |
| `$in` / `$nin` | Pertenece / no pertenece a una lista | `{ categoria: { $in: ['audio', 'movil'] } }` |

```jsx
db.productos.find({ precio: { $gte: 50, $lte: 200 } })
```

### Operadores lógicos

| Operador | Función |
| --- | --- |
| `$and` | Deben cumplirse todas las condiciones. |
| `$or` | Basta con una. |
| `$nor` | No debe cumplirse ninguna. |
| `$not` | Niega una condición **de un campo**. |

```jsx
// precio menor que 50 O stock mayor que 20
db.productos.find({ $or: [{ precio: { $lt: 50 } }, { stock: { $gt: 20 } }] })

// $not va dentro del campo, no en el nivel superior
db.productos.find({ precio: { $not: { $gt: 100 } } })
```

**Error típico:** `{ $not: { precio: { $gt: 100 } } }` da error. `$not` siempre se escribe dentro del campo.

### Campos y tipos

| Operador | Función | Ejemplo |
| --- | --- | --- |
| `$exists` | ¿Existe el campo? | `{ descuento: { $exists: true } }` |
| `$type` | Filtra por tipo BSON | `{ precio: { $type: 'number' } }` |
| `$regex` | Expresión regular | `{ nombre: { $regex: 'rat', $options: 'i' } }` |
| `$expr` | Compara campos entre sí | `{ $expr: { $gt: ['$precio', '$coste'] } }` |

La opción `i` ignora mayúsculas, **pero no tildes**: buscar `'raton'` no encuentra `'Ratón'`.

### Ordenar y paginar

```jsx
db.productos.find().sort({ precio: 1 }).limit(3)               // los 3 más baratos
db.productos.find().sort({ precio: -1 })                       // de mayor a menor
db.productos.find().sort({ _id: 1 }).skip(3).limit(3)          // página 2 de tamaño 3
```

`1` es ascendente y `-1` descendente. Para una paginación estable, ordena por un campo único (por ejemplo `_id`).

## 7. Actualizar (UPDATE)

| Comando / operador | Función |
| --- | --- |
| `updateOne(filtro, cambio)` | Modifica el primer documento que coincide. |
| `updateMany(filtro, cambio)` | Modifica todos los que coinciden. |
| `replaceOne(filtro, doc)` | Sustituye el documento entero (menos `_id`). |
| `$set` / `$unset` | Asigna o crea un campo / lo elimina. |
| `$inc` / `$mul` | Suma o resta / multiplica un número. |
| `$rename` | Cambia el nombre de un campo. |
| `$min` / `$max` | Modifica solo si el nuevo valor es menor / mayor. |
| `$currentDate` | Asigna la fecha actual. |
| `$setOnInsert` | Asigna valores solo si un `upsert` inserta. |

```jsx
db.productos.updateOne({ nombre: 'Ratón' }, { $set: { precio: 25 } })
db.productos.updateOne({ nombre: 'Ratón' }, { $inc: { stock: -1 } })

// upsert: actualiza si existe, inserta si no
db.productos.updateOne({ codigo: 'P010' }, { $set: { nombre: 'Alfombrilla', precio: 8 } }, { upsert: true })
```

## 8. Eliminar (DELETE)

| Comando | Función |
| --- | --- |
| `deleteOne({ codigo: 'P010' })` | Elimina un documento. |
| `deleteMany({ stock: 0 })` | Elimina todos los que coinciden. |
| `deleteMany({})` | Vacía la colección, pero la conserva. |
| `drop()` | Elimina la colección completa. |

<aside>
💡

Antes de un borrado masivo, **ejecuta primero el mismo filtro con `find()`** para comprobar qué vas a eliminar.

</aside>

## 9. Arrays y documentos anidados

Prepara un array de documentos para los ejemplos:

```jsx
db.productos.updateOne({ codigo: 'P001' }, { $set: { valoraciones: [{ usuario: 'ana', nota: 5 }, { usuario: 'luis', nota: 3 }] } })
```

### Consultar

| Operador | Función | Ejemplo |
| --- | --- | --- |
| Valor simple | El array contiene ese valor | `{ etiquetas: 'gaming' }` |
| `$all` | Contiene todos los valores | `{ etiquetas: { $all: ['gaming', 'oficina'] } }` |
| `$size` | Longitud exacta | `{ etiquetas: { $size: 2 } }` |
| `$elemMatch` | Un mismo elemento cumple varias condiciones | ver abajo |
| Notación con punto | Campo de un documento anidado | `{ 'valoraciones.usuario': 'ana' }` |

```jsx
// Un mismo elemento con usuario ana Y nota >= 4
db.productos.find({ valoraciones: { $elemMatch: { usuario: 'ana', nota: { $gte: 4 } } } })
```

`$elemMatch` evita un error muy común: sin él, `{ 'valoraciones.usuario': 'luis', 'valoraciones.nota': 5 }` se cumple aunque `luis` tenga un 3 y otro usuario tenga un 5, porque cada condición se evalúa por separado.

### Modificar arrays

| Operador | Función |
| --- | --- |
| `$push` | Añade un elemento. |
| `$addToSet` | Añade el elemento solo si no existe. |
| `$pull` | Elimina los elementos que coinciden. |
| `$pop` | Elimina el primero (`-1`) o el último (`1`). |
| `$each` | Añade varios elementos con `$push` o `$addToSet`. |

```jsx
db.productos.updateOne({ codigo: 'P001' }, { $push: { etiquetas: 'oferta' } })
db.productos.updateOne({ codigo: 'P001' }, { $addToSet: { etiquetas: { $each: ['oferta', 'nuevo'] } } })
db.productos.updateOne({ codigo: 'P001' }, { $pull: { etiquetas: 'nuevo' } })
```

### Modificar un elemento concreto del array

```jsx
// $ : el primer elemento que coincide con el filtro
db.productos.updateOne({ codigo: 'P001', 'valoraciones.usuario': 'luis' }, { $set: { 'valoraciones.$.nota': 4 } })

// $[] : todos los elementos
db.productos.updateOne({ codigo: 'P001' }, { $inc: { 'valoraciones.$[].nota': 1 } })

// $[v] con arrayFilters : los elementos que cumplan una condición
db.productos.updateOne({ codigo: 'P001' }, { $set: { 'valoraciones.$[v].nota': 5 } }, { arrayFilters: [{ 'v.nota': { $lt: 4 } }] })
```

## 10. Modelado: ¿embeber o referenciar?

Es la decisión que más cambia respecto a SQL. En lugar de normalizar siempre, en MongoDB se diseñan los documentos según **cómo se van a consultar**.

|  | Embeber (dentro del documento) | Referenciar (guardar el `_id` de otro documento) |
| --- | --- | --- |
| Cuándo | Relación 1 a pocos; los datos se leen siempre juntos | Relación 1 a muchos; los datos se usan por separado o se comparten |
| Ejemplo | Las `lineas` de un pedido; las `valoraciones` de un producto | El `clienteId` de un pedido |
| Ventajas | Una sola consulta, escritura atómica de un documento | Sin duplicación, documentos pequeños |
| Inconvenientes | Documentos grandes (límite de 16 MB), datos duplicados | Hay que usar `$lookup` o dos consultas |

<aside>
💡

Regla práctica: si los datos **nacen y mueren juntos** y se leen juntos, embébelos. Si crecen sin límite o se comparten, referencia.

</aside>

## 11. Agregaciones (`aggregate`) — Ejemplos ampliados

Una agregación es un pipeline: una secuencia de etapas que filtran, agrupan, calculan o transforman documentos.

Para los ejemplos utilizaremos una base de datos de un centro educativo, con las colecciones `alumnos`, `cursos` y `matriculas`.

### Datos para practicar

Ejecuta una vez estos comandos en `mongosh` sobre una base de datos de pruebas.

```jsx
use instituto

db.alumnos.insertMany([
  { _id: 1, nombre: "Ana", ciudad: "Madrid", edad: 20,
    notas: [8, 9, 7], lenguajes: ["Java", "Python"] },
  { _id: 2, nombre: "Luis", ciudad: "Toledo", edad: 23,
    notas: [5, 6, 7], lenguajes: ["Java"] },
  { _id: 3, nombre: "Marta", ciudad: "Madrid", edad: 21,
    notas: [9, 9, 10], lenguajes: ["Python", "JavaScript"] },
  { _id: 4, nombre: "Pablo", ciudad: "Getafe", edad: 19,
    notas: [4, 5, 6], lenguajes: ["Java", "JavaScript"] },
  { _id: 5, nombre: "Lucía", ciudad: "Toledo", edad: 25,
    notas: [7, 8, 8], lenguajes: ["Python"] }
])

db.cursos.insertMany([
  { _id: 101, nombre: "Acceso a Datos", horas: 165 },
  { _id: 102, nombre: "Interfaces", horas: 120 },
  { _id: 103, nombre: "Servicios", horas: 100 }
])

db.matriculas.insertMany([
  { alumnoId: 1, cursoId: 101, nota: 9 },
  { alumnoId: 1, cursoId: 102, nota: 8 },
  { alumnoId: 2, cursoId: 101, nota: 6 },
  { alumnoId: 3, cursoId: 101, nota: 10 },
  { alumnoId: 3, cursoId: 103, nota: 9 },
  { alumnoId: 4, cursoId: 102, nota: 5 },
  { alumnoId: 5, cursoId: 103, nota: 8 }
])
```

### 1. `$match` — Filtrar documentos

Devuelve únicamente los alumnos de Madrid mayores de 20 años.

```jsx
db.alumnos.aggregate([
  { $match: {
      ciudad: "Madrid",
      edad: { $gt: 20 }
  }}
])
```

### 2. `$group` — Agrupar documentos

Cuenta los alumnos de cada ciudad y calcula su edad media.

```
db.alumnos.aggregate([
  { $group: {
      _id: "$ciudad",
      total: { $sum: 1 },
      edadMedia: { $avg: "$edad" }
  }}
])
```

### 3. `$sort`, `$limit` y `$skip` — Ordenar y paginar

Obtiene el segundo y tercer alumno más joven.

```jsx
db.alumnos.aggregate([
  { $sort: { edad: 1, _id: 1 } },
  { $skip: 1 },
  { $limit: 2 }
])
```

### 4. `$project` — Seleccionar y calcular campos

Muestra el nombre, la edad y la edad que tendrán el próximo año.

```jsx
db.alumnos.aggregate([
  { $project: {
      _id: 0,
      nombre: 1,
      edadActual: "$edad",
      edadProxima: { $add: ["$edad", 1] }
  }}
])
```

### 5. `$addFields` / `$set` — Añadir campos

Calcula la nota media de cada alumno sin modificar los documentos almacenados.

```jsx
db.alumnos.aggregate([
  { $addFields: {
      notaMedia: { $avg: "$notas" }
  }}
])
```

### 6. `$unset` — Eliminar campos del resultado

Devuelve los alumnos sin mostrar su edad ni sus notas.

```jsx
db.alumnos.aggregate([
  { $unset: ["edad", "notas"] }
])
```

No modifica los datos guardados.

### 7. `$unwind` — Descomponer arrays

Transforma cada lenguaje de programación de un alumno en un documento independiente.

```jsx
db.alumnos.aggregate([
  { $unwind: "$lenguajes" },
  { $project: {
      _id: 0,
      nombre: 1,
      lenguaje: "$lenguajes"
  }}
])
```

Un alumno que conozca dos lenguajes aparecerá en dos resultados.

### 8. `$sortByCount` — Contar frecuencias

Obtiene los lenguajes más conocidos entre los alumnos.

```jsx
db.alumnos.aggregate([
  { $unwind: "$lenguajes" },
  { $sortByCount: "$lenguajes" }
])
```

### 9. `$count` — Contar documentos

Cuenta cuántos alumnos tienen 21 años o más.

```jsx
db.alumnos.aggregate([
  { $match: { edad: { $gte: 21 } } },
  { $count: "alumnosMayores" }
])
```

### 10. `$bucket` — Agrupar por intervalos

Clasifica al alumnado en grupos de edad.

```jsx
db.alumnos.aggregate([
  { $bucket: {
      groupBy: "$edad",
      boundaries: [18, 21, 24, 30],
      default: "Otros",
      output: {
        total: { $sum: 1 },
        nombres: { $push: "$nombre" }
      }
  }}
])
```

Los intervalos incluyen el límite inferior y excluyen el superior.

### 11. `$facet` — Varias estadísticas a la vez

Obtiene el número de alumnos por ciudad y las edades mínima, máxima y media en una misma consulta.

```jsx
db.alumnos.aggregate([
  { $facet: {
      porCiudad: [
        { $sortByCount: "$ciudad" }
      ],
      edades: [
        { $group: {
            _id: null,
            minima: { $min: "$edad" },
            maxima: { $max: "$edad" },
            media: { $avg: "$edad" }
        }}
      ]
  }}
])
```

### 12. `$lookup` — Relacionar colecciones

Muestra cada matrícula junto con los datos del alumno.

```jsx
db.matriculas.aggregate([
  { $lookup: {
      from: "alumnos",
      localField: "alumnoId",
      foreignField: "_id",
      as: "alumno"
  }}
])
```

El campo `alumno` será un array con los documentos relacionados.

### 13. `$lookup` + `$unwind` — Mostrar relaciones

Obtiene el nombre del alumno, el curso y su nota.

```jsx
db.matriculas.aggregate([
  { $lookup: {
      from: "alumnos",
      localField: "alumnoId",
      foreignField: "_id",
      as: "alumno"
  }},
  { $unwind: "$alumno" },
  { $lookup: {
      from: "cursos",
      localField: "cursoId",
      foreignField: "_id",
      as: "curso"
  }},
  { $unwind: "$curso" },
  { $project: {
      _id: 0,
      alumno: "$alumno.nombre",
      curso: "$curso.nombre",
      nota: 1
  }}
])
```

### 14. `$out` — Guardar resultados en otra colección

Crea una colección `resumen_ciudades` con el número de alumnos por ciudad.

```jsx
db.alumnos.aggregate([
  { $group: {
      _id: "$ciudad",
      total: { $sum: 1 }
  }},
  { $out: "resumen_ciudades" }
])
```

Atención: si la colección de destino existe, `$out` reemplaza sus datos con el resultado de la agregación.

### 15. `$merge` — Integrar resultados

Actualiza o inserta estadísticas en una colección de destino, en lugar de reemplazarla completamente.

```jsx
db.alumnos.aggregate([
  { $group: {
      _id: "$ciudad",
      total: { $sum: 1 }
  }},
  { $merge: {
      into: "estadisticas_ciudades",
      on: "_id",
      whenMatched: "merge",
      whenNotMatched: "insert"
  }}
])
```

En este ejemplo, los documentos existentes de otras ciudades no se eliminan.

## 11.1. Acumuladores y operadores de cálculo

Además de las etapas, conviene que el alumnado practique los operadores que se utilizan dentro de ellas.

| Operador | Función | Ejemplo de uso |
| --- | --- | --- |
| `$sum` | Sumar valores | Total de horas matriculadas |
| `$avg` | Calcular medias | Nota media por curso |
| `$min` / `$max` | Mínimo y máximo | Nota más baja y más alta |
| `$first` / `$last` | Primer y último valor del grupo | Alumno más joven y más mayor, ordenando previamente |
| `$push` | Agrupar valores en un array | Nombres de alumnos por ciudad |
| `$addToSet` | Agrupar valores sin repetidos | Cursos distintos por alumno |
| `$multiply` | Multiplicar | Calcular puntuaciones ponderadas |
| `$subtract` | Restar | Diferencia entre dos notas |
| `$divide` | Dividir | Calcular porcentajes |
| `$concat` | Concatenar cadenas | Crear etiquetas descriptivas |
| `$cond` | Condición similar a `if/else` | Aprobado o suspenso |
| `$ifNull` | Sustituir valores nulos o ausentes | Mostrar cero cuando falta una nota |

### Ejemplo: aprobados y suspensos con `$cond`

```jsx
db.matriculas.aggregate([
  { $project: {
      _id: 0,
      alumnoId: 1,
      nota: 1,
      resultado: {
        $cond: [
          { $gte: ["$nota", 5] },
          "APROBADO",
          "SUSPENSO"
        ]
      }
  }}
])
```

### Ejemplo: estadísticas de cada curso

```jsx
db.matriculas.aggregate([
  { $group: {
      _id: "$cursoId",
      notaMedia: { $avg: "$nota" },
      notaMaxima: { $max: "$nota" },
      notaMinima: { $min: "$nota" },
      matriculados: { $sum: 1 }
  }},
  { $sort: { notaMedia: -1 } }
])
```

### Ejemplo: alumnos agrupados por ciudad

```jsx
db.alumnos.aggregate([
  { $group: {
      _id: "$ciudad",
      alumnos: { $push: "$nombre" },
      total: { $sum: 1 }
  }}
])
```

## 12. Relacionar colecciones con `$lookup`

Utilizaremos la base de datos `instituto` del apartado 11, que contiene tres colecciones:

- `alumnos`: `_id`, `nombre`, `ciudad`, `edad`, `notas`, `lenguajes`.
- `cursos`: `_id`, `nombre`, `horas`.
- `matriculas`: `alumnoId`, `cursoId`, `nota`.

### Ejemplo 1. Cada matrícula con los datos del alumno

```jsx
db.matriculas.aggregate([
  {
    $lookup: {
      from: "alumnos",
      localField: "alumnoId",
      foreignField: "_id",
      as: "alumno"
    }
  },
  { $unwind: "$alumno" }
])
```

`$lookup` busca los alumnos relacionados con cada matrícula y los incorpora en un array llamado `alumno`.

`$unwind` descompone ese array. Cuando contiene un único documento, permite trabajar con él como objeto anidado.

### Ejemplo 2. Mostrar el nombre del alumno, el curso y la nota

```jsx
db.matriculas.aggregate([
  {
    $lookup: {
      from: "alumnos",
      localField: "alumnoId",
      foreignField: "_id",
      as: "alumno"
    }
  },
  { $unwind: "$alumno" },
  {
    $lookup: {
      from: "cursos",
      localField: "cursoId",
      foreignField: "_id",
      as: "curso"
    }
  },
  { $unwind: "$curso" },
  {
    $project: {
      _id: 0,
      alumno: "$alumno.nombre",
      curso: "$curso.nombre",
      nota: 1
    }
  }
])
```

Resultado de ejemplo:

```
{
  "alumno": "Ana",
  "curso": "Acceso a Datos",
  "nota": 9
}
```

### Ejemplo 3. Calcular la nota media de cada curso

```jsx
db.matriculas.aggregate([
  {
    $group: {
      _id: "$cursoId",
      notaMedia: { $avg: "$nota" },
      matriculados: { $sum: 1 }
    }
  },
  {
    $lookup: {
      from: "cursos",
      localField: "_id",
      foreignField: "_id",
      as: "curso"
    }
  },
  { $unwind: "$curso" },
  {
    $project: {
      _id: 0,
      curso: "$curso.nombre",
      notaMedia: 1,
      matriculados: 1
    }
  }
])
```

Combina `$group`, `$lookup`, `$unwind` y `$project` para generar un informe.

## 13. Índices y rendimiento

Los índices ayudan a localizar documentos sin tener que recorrer toda la colección.

| Comando | Función |
| --- | --- |
| `createIndex({ ciudad: 1 })` | Índice ascendente por ciudad. |
| `createIndex({ ciudad: 1, edad: -1 })` | Índice compuesto por ciudad y edad. |
| `createIndex({ nombre: 1 }, { unique: true })` | Impide nombres duplicados en la colección. |
| `getIndexes()` | Lista los índices existentes. |
| `dropIndex("ciudad_1")` | Elimina un índice concreto. |
| `find(...).explain("executionStats")` | Muestra el plan y estadísticas de ejecución. |

### Ejemplo: consultar antes y después de crear un índice

1. Consultar sin crear un índice adicional

```jsx
db.alumnos.find({
  ciudad: "Madrid"
}).explain("executionStats")
```

2. Crear un índice

```jsx
db.alumnos.createIndex({ ciudad: 1 })
```

3. Repetir la consulta

```jsx
db.alumnos.find({
  ciudad: "Madrid"
}).explain("executionStats")
```

### Cómo interpretar `explain()`

| Indicador | Significado |
| --- | --- |
| `COLLSCAN` | Escaneo completo de la colección. |
| `IXSCAN` | Escaneo de un índice. |
| `totalDocsExamined` | Documentos examinados. |
| `totalKeysExamined` | Entradas de índice examinadas. |
| `nReturned` | Documentos devueltos. |
| `executionTimeMillis` | Tiempo de ejecución registrado. |

Con solo cinco alumnos, MongoDB puede decidir que recorrer toda la colección es más eficiente que utilizar el índice. No hay que asumir que aparecerá `IXSCAN` ni que será más rápido.

### Ejemplo: índice compuesto

```jsx
db.alumnos.createIndex({
  ciudad: 1,
  edad: -1
})

db.alumnos.find({
  ciudad: "Madrid"
}).sort({ edad: -1 })
```

Este índice está pensado para búsquedas por ciudad con ordenación descendente por edad.

### Ejemplo: índice único

```jsx
db.cursos.createIndex(
  { nombre: 1 },
  { unique: true }
)

// Intentamos insertar un curso duplicado
db.cursos.insertOne({
  _id: 104,
  nombre: "Acceso a Datos",
  horas: 165
})
```

La inserción falla porque ya existe un curso con ese nombre y el índice exige valores únicos.

Los índices mejoran determinadas consultas, pero ocupan espacio y añaden trabajo a las escrituras.

## 14. Validación de documentos

MongoDB permite establecer reglas para controlar qué documentos se pueden insertar o modificar.

Utilizaremos una nueva colección llamada `profesores`, dentro de `instituto`.

### Ejemplo 1. Crear una colección con validación

```jsx
db.createCollection("profesores", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["nombre", "email"],
      properties: {
        nombre: {
          bsonType: "string"
        },
        email: {
          bsonType: "string",
          pattern: "^.+@.+$"
        },
        edad: {
          bsonType: "number",
          minimum: 18,
          maximum: 70
        },
        departamento: {
          enum: ["Informatica", "Administracion", "Idiomas"]
        }
      }
    }
  },
  validationAction: "error"
})
```

### Ejemplo 2. Insertar un documento válido

```jsx
db.profesores.insertOne({
  nombre: "Carlos",
  email: "carlos@instituto.es",
  edad: 35,
  departamento: "Informatica"
})
```

La inserción se realiza correctamente.

### Ejemplo 3. Insertar documentos inválidos

```jsx
// ERROR: falta el campo obligatorio email
db.profesores.insertOne({
  nombre: "Laura",
  edad: 30
})
```

```jsx
// ERROR: edad fuera del intervalo permitido
db.profesores.insertOne({
  nombre: "Pedro",
  email: "pedro@instituto.es",
  edad: 15
})
```

```jsx
// ERROR: departamento no permitido
db.profesores.insertOne({
  nombre: "Marta",
  email: "marta@instituto.es",
  departamento: "Matematicas"
})
```

### Ejemplo 4. Modificar la validación con `collMod`

Añadimos el campo obligatorio `especialidad`.

```jsx
db.runCommand({
  collMod: "profesores",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["nombre", "email", "especialidad"],
      properties: {
        nombre: { bsonType: "string" },
        email: {
          bsonType: "string",
          pattern: "^.+@.+$"
        },
        edad: {
          bsonType: "number",
          minimum: 18,
          maximum: 70
        },
        departamento: {
          enum: ["Informatica", "Administracion", "Idiomas"]
        },
        especialidad: { bsonType: "string" }
      }
    }
  },
  validationAction: "error"
})
```

`collMod` sustituye la configuración anterior del validador por la nueva. Por eso repetimos las reglas que queremos conservar.

Importante: la nueva validación se aplica a las inserciones y actualizaciones posteriores; no elimina automáticamente los documentos existentes que incumplan las nuevas reglas.

### Elementos de validación

| Elemento | Función |
| --- | --- |
| `required` | Define los campos obligatorios. |
| `properties` | Establece las reglas de cada campo. |
| `bsonType` | Comprueba el tipo de dato. |
| `minimum` / `maximum` | Establece límites numéricos. |
| `pattern` | Comprueba un texto mediante una expresión regular. |
| `enum` | Restringe los valores permitidos. |
| `validationAction: "error"` | Rechaza documentos que incumplen las reglas. |
| `validationAction: "warn"` | Permite la operación, pero registra una advertencia. |
| `collMod` | Modifica la validación de una colección existente. |