# Procesos de compra hechos en la plataforma SECOP II - Enero a Junio del 2025

## Descripción general

Este proyecto analiza los procesos de compra pública realizados en la plataforma SECOP II durante el primer semestre de 2025, estén o no adjudicados. El objetivo es explorar, modelar y visualizar la información para obtener insights relevantes sobre la contratación estatal en Colombia.

*SECOP II, creado por la Ley 1150 de 2007, es el punto único de ingreso de información y generación de reportes para entidades estatales y la ciudadanía en materia de compra y contratación pública. Los datos son actualizados diariamente por Colombia Compra Eficiente y publicados en el portal de Datos Abiertos.*

---

## Herramientas utilizadas

- Power BI Desktop
- Power BI Services

---

## Información del conjunto de datos

- **Fuente:** [Datos Abiertos Colombia – SECOP II](https://www.datos.gov.co/Gastos-Gubernamentales/SECOP-II-Procesos-de-Contrataci-n/p6dx-8zbt/about_data)
- **Nombre:** SECOP II - Procesos de Contratación
- **Categoría:** Gastos Gubernamentales
- **Descripción:** Registro de los procesos de compra, sean o no adjudicados, hechos en la plataforma SECOP II  desde su lanzamiento
- **Fecha de actualización:** 1 de julio de 2025
- **Datos suministrados por:** Colombia Compra Eficiente
- **Propietario:** Colombia Compra Eficiente (Datos Abiertos CCE)
- **Filtrado aplicado:** Solo procesos publicados entre el 01 de enero y el 30 de junio de 2025
- **Filas analizadas:** 1.014.863 donde cada fila es un proceso
- **Columnas originales:** 59
- **Columnas seleccionadas para análisis:** 22

---

# Análisis exploratorio de datos

## Selección de datos

- Se exploran las 59 columnas originales y se seleccionan 22 relevantes para el análisis.
- Las 37 columnas restantes se eliminan por ser irrelevantes, incompletas o redundantes.

**Columnas seleccionadas:**
- Entidad
- Departamento Entidad
- Ciudad Entidad
- OrdenEntidad
- Entidad Centralizada
- ID del Proceso
- PCI
- Fecha de Publicación del Proceso
- Modalidad de Contratación
- Nombre de la Unidad de Contratación
- Estado del Procedimiento
- Adjudicado
- ID Adjudicación
- Código Proveedor
- Departamento Proveedor
- Ciudad Proveedor
- Valor Total Adjudicación
- Nombre del Proveedor Adjudicado
- Código Principal de Categoría
- Estado de Apertura del Proceso
- Tipo de Contrato
- Estado Resumen

## Validación de tipos de datos

- Se identifican y corrigen problemas de formato regional en fechas y datos numéricos.
- Se ajustan algunos tipos de datos

---

## Diseño del modelo de datos

- Se define un modelo estrella con una tabla de hechos de los procesos y 10 tablas de dimensiones (Entidad, Proveedor, Adjudicado,Estado Apertura, Estado procedimiento, Fase, Modelidad de contratación, Tipo Contrato, Segmento y fecha.
- Se crean claves primarias y foráneas, incluidas claves sustitutas generadas manualmente cuando no existen en el origen.

![Diseño](images/Diseño%20del%20modelo.png)
---

# Proceso de ETL (Extracción, Transformación y Carga)

## 1. Carga inicial

- Se importa el dataset original a Power Query.
- Se eliminan columnas innecesarias y se inhabilita la carga de la tabla base en el modelo.

  ![Dateset](images/Original.png)

## 2. Creación de tablas de dimensiones

Para cada dimensión:
- Crear tabla de referencia desde la tabla principal.
- Seleccionar la columna relevante y quitar el resto.
- Generar perfiles según el dataset completo.
- Quitar duplicados.
- Crear clave primaria (índice si aplica).
- Renombrar  Columnas
- Ajustar tipos de datos.
- Limpiar caracteres especiales y espacios.

![Dim_Adjudicado](images/Adjudicado.png)
![Dim_Apertura](images/Apertura.png)
![Dim_Contrato](images/Contrato.png)
![Dim_Entidad](images/Entidad.png)
![Dim_Fase](images/Fase.png)
![Dim_Modalidad](images/Modalidad.png)
![Dim_Procedimiento](images/Procedimiento.png)
![Dim_Proveedor](images/Proveedor.png)

## 3. Creación de la tabla de hechos

- Se crea por referencia desde la tabla principal.
- Incluye solo: ID del Proceso, Fecha de Publicación, ID Adjudicación, Valor Total Adjudicación y claves foráneas.
- Se realizan combinaciones externas izquierdas (merge) con cada dimensión para traer las claves.
- Se crea una columna personalizada para el ID de segmento (primeros dos dígitos del código de categoría principal).
- Se eliminan columnas innecesarias
- se ajustan tipos de datos.

![Hechos](images/Hechos.png)

## 4. Dimensión segmento
*The United Nations Standard Products and Services Code® - UNSPSC - Código Estándar de Productos y Servicios de Naciones Unidas, es una metodología uniforme de codificación utilizada para clasificar productos y servicios fundamentada en un arreglo jerárquico y en una estructura lógica. Este sistema de clasificación permite codificar productos y servicios de forma clara ya que se basa en estándares acordados por la industria los cuales facilitan el comercio entre empresas y gobierno.*

- Se añade un nuevo origen de datos: lista de códigos UNSPSC.
- Crear Tabla de Referencia a la tabla de hechos y se crea la dimensión segmento a partir del ID de segmento extraído
- Se asocia el nombre del segmento desde el archivo UNSPSC mediante combinacion externa izquierdas (merge)
- Se eliminan duplicados
- Se inhabilita la carga del archivo UNSPSC en el modelo.

![Dim_Segmento](images/Segmento.png)

## Resultado Final

![Final_Query](images/Final_Query.png)

## 5. Tabla de fechas

- Se crea mediante DAX (`CALENDAR`) con el rango 01/01/2025 a 30/06/2025.
- Se añaden columnas de año, número de mes, día y nombre de mes.
- Se deshabilita la fecha y hora automáticas y se marca como tabla de fechas.
- Se ordena la columna de mes por el número de mes.

![Dim_Fechas](images/Fechas.png)

---

# Modelado de datos

- Se establecen relaciones varios a uno entre la tabla de hechos y cada dimensión.
- Se anclan campos relacionados en la parte superior de las tarjetas.
- Se ocultan columnas de clave en el modelo para simplificar la vista.

![Modelo](images/Modelo.png)
---

## Medidas creadas

- **Procesos Adjudicados:** Indica el numero de procesos adjudicados o cero si no hay ninguno. 
  `Procesos Adjudicados = COALESCE(CALCULATE(COUNTROWS('Hechos_Procesos'), 'Hechos_Procesos'[ID_Adjudicado] = 1),0)`
- **Procesos de Compra:** Indica el numero de procesos totales. 
  `Procesos de Compra = COUNTROWS('Hechos_Procesos')`
- **Tasa de Adjudicación:** Muestra el % de adjudicación o cero si no hay ninguno.
  `Tasa de Adjudicación = COALESCE(DIVIDE([Procesos Adjudicados], [Procesos de Compra]),0)`
- **Valor Total Adjudicado:** Totaliza el valor adjudicado.
  `Valor Total Adjudicado = SUM('Hechos_Procesos'[Valor Total Adjudicacion])`

*La columna Valor Total Adjudicado se oculta en la tabla de hechos para evitar redundancia.*

![Dim_Medidas](images/Medidas.png)
---

# Creación del informe

El informe se divide en dos secciones principales:

## 1. Información general de procesos de compra
- Tarjetas
- Gráficos resumen de procesos por entidad, modalidad, estado, etc.
- Segmentador sincronizado por fecha.

  ![General](images/General.png)

## 2. Procesos adjudicados
- Foco en los procesos con adjudicación.
- KPI: para medir el avance en los procesos adjudicados sobre el total de procesos de compra
- Tarjetas
- Gráficos resumen por proveedor, segmento, valor adjudicado, etc
- Segmentadores sincronizados por ID de proceso y entidad.

![Adjudicados](images/Adjudicados.png)

## Funcionalidades adicionales

- **Drillthrough (Obtención de detalles):** Dos páginas ocultas para detalle de procesos de compra y de valor adjudicado, accesibles desde los gráficos principales.
- **Tooltips (Información sobre la herramienta)** al pasar el cursor sobre el gráfico de procesos por orden de entidad, se muestra detalle por entidad.
- **Visualizaciones variadas:** Barras, columnas, circulares, líneas, árbol, tablas y medidor.
- **Tema personalizado:** Colores corporativos de Colombia Compra Eficiente (amarillo, azul, rojo, blanco y negro).

---

## Publicación y panel en Power BI Services

- El informe se publica en Power BI Services.
  
![Services](images/Services.png)
  
- Se crea un panel para monitorear las métricas clave del proceso de adjudicación.

 ![Panel](images/Panel.png)
 
---

# Resultado final

- Modelo Semantico
- Informe
- Panel

 ![Resultado](images/Resultado.png)
---

# Notas y consideraciones

- Este proyecto es solo para fines demostrativos y de portafolio.
- Los datos usados son de dominio público
- Si necesitas abrir el archivo PBIX:
  1. Descárgalo desde este repositorio.
  2. Ábrelo en Power BI Desktop


