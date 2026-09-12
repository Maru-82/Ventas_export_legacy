# Ventas_export_legacy

Documentación del proceso de limpieza y transformación del archivo `Ventas_export_legacy.xlsx` en Power Query, para el ejercicio de Conectividad y Transformación de Datos en Power BI.

## Carga del archivo

Se cargó el archivo desde Inicio > Obtener datos > Excel, se seleccionó la hoja `VENTAS_EXPORT` y se usó "Transformar datos" para entrar directo a Power Query.

## Orden de las transformaciones

Se siguió el mismo orden del enunciado: primero se renombraron las columnas para poder identificar qué tenía cada una, después se corrigieron los tipos de datos, se limpiaron duplicados y nulos, y por último se separó la información en dos tablas.

### 1. Renombrado de columnas

Se cambiaron los 20 nombres técnicos por nombres descriptivos en español y sin tildes:

| Original | Nuevo |
|---|---|
| COD_OP | id_operacion |
| COD_CLI | id_cliente |
| NOM_CLI | nombre_cliente |
| MAIL_CLI | email_cliente |
| TEL_CLI | telefono_cliente |
| CIU_CLI | ciudad_cliente |
| PROV_CLI | provincia_cliente |
| SEG_CLI | segmento_cliente |
| FLG_ACT | cliente_activo |
| F_ALTA_CLI | fecha_alta_cliente |
| F_VTA | fecha_venta |
| COD_PROD | id_producto |
| DESC_PROD | nombre_producto |
| RUBRO_PROD | categoria_producto |
| CANT | cantidad |
| PU_VTA | precio_unitario |
| DTO_PCT | descuento |
| TOT_VTA | total_vta (se modifica en otro paso posteior) |
| COD_MON | moneda |
| CANAL_VTA | canal_venta |

**FLG_ACT → cliente_activo:** el archivo no trae ningún diccionario de datos que confirme qué significa esta columna. Se asumió que es una bandera de cliente activo por la abreviatura (FLG = flag, ACT = activo) y porque los únicos valores son S/N.

### 2. Tipos de datos

- **nombre_cliente:** se aplicó Transformar > Formato > Recortar, porque varios nombres tenían espacios de más al final (ej. "Santiago Pereyra  ").
- **cliente_activo:** se reemplazó S por true y N por false, y se cambió el tipo a Verdadero/Falso
- **fecha_alta_cliente:** se usó Cambiar tipo > Usando configuración regional > Fecha > Español (Argentina), porque el formato original (DD/MM/YYYY) es ambiguo si no se especifica la configuración regional.
- **fecha_venta:** se usó Cambiar tipo > Usando configuración regional > Fecha > Español (Argentina), porque el formato original (DD/MM/YYYY) es ambiguo si no se especifica la configuración regional.
- **precio_unitario:** se pasó a Número decimal fijo, el tipo pensado para montos de dinero.
- **descuento:** primero se reemplazaron los nulos por 0 (ver siguiente sección), y después se cambió el tipo a Porcentaje en vez de Número decimal, porque representa un porcentaje de descuento.
- **canal_venta:** se aplicó Formato > Poner en mayúscula cada palabra, porque el mismo dato estaba escrito de formas distintas (SUCURSAL, Sucursal, sucursal, ONLINE, Online, online, TELEFONICO, Telefonico).

### 3. Duplicados y nulos

- Se eliminaron las filas completamente vacías con Quitar filas > Quitar filas en blanco.
- Se eliminaron las filas duplicadas con Quitar filas > Quitar duplicados, sobre todas las columnas.
- **descuento (nulos):** se reemplazaron por 0, asumiendo que un descuento vacío significa que no se aplicó descuento, no que falte el dato.
- **total_vta (nulos):** se recalcularon con una columna personalizada:
  ```
  if [TOT_VTA] = null then [cantidad] * [precio_unitario] * (1 - [descuento]) else [TOT_VTA]
  ```
  Se renombra a total_venta 

### 4. Separación en dos tablas

**CLIENTES** (una fila por cliente): id_cliente, nombre_cliente, email_cliente, telefono_cliente, ciudad_cliente, provincia_cliente, segmento_cliente, cliente_activo, fecha_alta_cliente.

**TRANSACCIONES** (una fila por venta): id_operacion, id_cliente, fecha_venta, id_producto, nombre_producto, categoria_producto, cantidad, precio_unitario, descuento, total_vta, moneda, canal_venta.

Las dos tablas se armaron como referencia de la consulta original, no como duplicado, para que si se corrige algo de la limpieza base más adelante, el cambio se propague solo a las dos tablas. Se deshabilitó la carga de la consulta original (VENTAS_EXPORT) al modelo, porque toda su información ya quedó repartida entre CLIENTES y TRANSACCIONES.

**Posible mejora:** con el mismo criterio, nombre_producto y categoria_producto también se repiten cada vez que se vende el mismo producto, así que se podría separar una tercera tabla PRODUCTOS. No se armó en esta entrega porque el enunciado pide separar en dos tablas (cliente y transacción), no tres.
