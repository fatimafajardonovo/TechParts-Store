# Ejercicio Propuesto: "TechParts Store" (Gestión de Inventario de Componentes)

> **Objetivo:** Diseñar un modelo de datos en Mongoose con validaciones robustas, poblar una colección en **MongoDB Atlas** mediante un script de semilla, ejecutar consultas avanzadas y exponer un endpoint en **Vercel** que permita filtrar el inventario.

---

## Contexto del Negocio

Una tienda de componentes de hardware para PC llamada *TechParts* necesita digitalizar su catálogo. Actualmente sufren porque los empleados ingresan precios negativos, códigos duplicados o categorías inventadas por error. Tu misión es construir la capa de datos utilizando Mongoose para blindar la base de datos y facilitar la búsqueda de productos.

---

## Tareas a Realizar

### Tarea 1: Crear el Modelo de Datos (`models/Producto.js`)

Crea un archivo llamado `models/Producto.js` que exporte un modelo de Mongoose llamado `Producto`. El esquema debe cumplir estrictamente con las siguientes reglas de validación:

1. **`codigo`**: `String`, obligatorio, único, en mayúsculas y con espacios limpios (`trim`). Ejemplo: `CPU-01`.
2. **`nombre`**: `String`, obligatorio, `trim`, con un largo mínimo de `4` caracteres.
3. **`categoria`**: `String`, obligatorio, restringido mediante un `enum` a los siguientes valores exactos:
   * `'Procesadores'`
   * `'Tarjetas Gráficas'`
   * `'Memorias RAM'`
   * `'Almacenamiento'`
4. **`precio`**: `Number`, obligatorio, con un valor mínimo de `0` (un producto no puede costar menos de 0).
5. **`stock`**: `Number`, obligatorio, número entero con valor mínimo de `0`.
6. **`compatibleCon`**: Un arreglo de strings (`[String]`) para indicar sockets o tecnologías compatibles (ej: `['LGA1700', 'DDR5']`). Su valor por defecto debe ser un arreglo vacío `[]`.
7. **`destacado`**: `Boolean`, con valor por defecto `false`.
8. **`timestamps`**: Actívalo para que cree automáticamente los campos `createdAt` y `updatedAt`.

*(No olvides añadir la protección para evitar recompilación del modelo en entornos serverless).*

---

### Tarea 2: Crear el Script de Población (`seed-productos.js`)

Crea un script en la raíz del proyecto para poblar la base de datos que realice lo siguiente:

1. Conectarse a MongoDB Atlas utilizando `dotenv` y tu variable `MONGODB_URI`.
2. Limpiar la colección existente (`Producto.deleteMany({})`).
3. Insertar un lote inicial de al menos **5 productos válidos** (combinando procesadores, tarjetas gráficas, memorias RAM y almacenamiento, con distintos precios, stock y estados de `destacado`).
4. **Prueba defensiva:** Añade un bloque `try/catch` interno que intente registrar a propósito un producto inválido (por ejemplo, con un precio negativo o una categoría fuera del `enum`) y muestra en la consola del terminal cómo Mongoose bloquea la inserción imprimiendo los errores por campo.

Ejecútalo en tu terminal:

```bash
node seed-productos.js
```

*(Comprueba en el panel web de MongoDB Atlas que los documentos se hayan guardado correctamente).*

---

### Tarea 3: Laboratorio de Consultas en Consola (`consultas-productos.js`)

Crea un archivo `consultas-productos.js` donde ejecutes y muestres en consola las siguientes **4 consultas** utilizando Mongoose:

1. **Consulta A:** Buscar todos los productos cuya categoría sea `'Tarjetas Gráficas'` y cuyo stock sea mayor a `0`.
2. **Consulta B:** Buscar componentes económicos cuyo precio sea menor o igual a `150`, utilizando proyección para mostrar únicamente el `nombre`, el `precio` y la `categoria` (excluyendo el `_id`).
3. **Consulta C:** Buscar productos que sean compatibles con una tecnología específica utilizando el operador `$in` (por ejemplo, buscar componentes que incluyan `'DDR5'` o `'AM5'` en su arreglo `compatibleCon`).
4. **Consulta D:** Obtener el **Top 3 de productos más caros** que estén marcados como `destacado: true`, ordenados de mayor a menor precio (`.sort({ precio: -1 })`), mostrando solo título y precio.

Ejecútalo en tu terminal:

```bash
node consultas-productos.js
```

---

### Tarea 4: Exponer el Endpoint Dinámico en Vercel (`api/productos.js`)

Crea una Serverless Function dentro de la carpeta `api/` llamada `productos.js` que permita consultar el inventario de forma pública filtrando mediante parámetros en la URL (`req.query`):

1. Debe permitir filtrar opcionalmente por **categoría** (`?categoria=Procesadores`).
2. Debe permitir filtrar opcionalmente por un **precio máximo** (`?precioMax=200`, usando el operador `$lte`).
3. Debe permitir filtrar opcionalmente por **compatibilidad** (`?compatibilidad=DDR5`, usando el operador `$in`).
4. Los resultados deben devolverse ordenados por precio ascendente (`.sort({ precio: 1 })`), junto con el conteo total de documentos encontrados.

Sube tu proyecto a Vercel, configura tu variable de entorno `MONGODB_URI` en el panel de control y prueba tus endpoints en el navegador:
* `https://tu-proyecto.vercel.app/api/productos`
* `https://tu-proyecto.vercel.app/api/productos?categoria=Tarjetas%20Gráficas`
* `https://tu-proyecto.vercel.app/api/productos?precioMax=300&compatibilidad=DDR5`

---

## Criterios de Evaluación / Validación para el Profesor

* [ ] **Esquema robusto:** Los alumnos usaron correctamente los validadores de Mongoose (`required`, `enum`, `min`, `unique`, `trim`).
* [ ] **Manejo de errores:** El script `seed` demuestra de forma práctica que Mongoose frena datos corruptos antes de llegar a la nube.
* [ ] **Dominio de Consultas:** Saben combinar `.find()`, operadores de comparación (`$lte`), operadores de arreglos (`$in`), proyecciones (`.select()`), ordenamiento (`.sort()`) y límites (`.limit()`).
* [ ] **Despliegue Serverless:** La API responde correctamente en Vercel interpretando los parámetros de la URL mediante `req.query`.
