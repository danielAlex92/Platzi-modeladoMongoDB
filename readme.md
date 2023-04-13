# Curso Mongo DB Modelado de Datos

- Mongo nos ofrece una alternativa a los diagramas entidad relación.
- Mongo tiene un maximo de 16MB para sus archivos de bases de datos


### Metodologia
- Requerimientos.
- Identificar ER.
- Aplicar patrones.
 
Cada una de estas fases se redea de ciertos aspectos.

- Escenario: Como serán los escenarios del modelo de negocio. (Ej: Como un usuario usará la app, donde podra editar sus mensajes o como comprará un producto)
- Expertos: Necesitamos expertos relacionados a nuestro tipo de negocio. (Ej: Si nos planeamos ofrecer un software orientado a la contabilidad, necesitamos a un experto, quien nos explique los conceptos que debemos manejar para su desarrollo.)
- Sistema actual: Tener en cuenta como funciona su sistema actual. Analizar como llevan a cabo sus tareas de la forma actual. (Ej: Las empresas llevan una gestión de sus ventas en un excel)
- DB Admin: El experto en modelado que une todas estas caracteristicas y llega a un resultado.
 
Este resultado serían 3 cosas:

- Workload: Donde indentificamos la carga de trabajo, las operaciones importantes, el tamaño de los datos, las consultas y posibles suposiciones.
- Relaciones: Lo obtenemos partir de a los sistemas actuales y el experto en modelado de datos. Identifcamos las entidades, atributos, restricciones y relaciones.
- Patrones: Cuando tenemos el diagrama de entidad-relación identificamos los patrones en el modelo de negocio que nos permiten realizar optimizaciónes de la carga de trabajo o obtener un mejor desempeño de la misma.
 
Todo esto nos lleva a un Diseño.

- IP: 0.0.0.0: Igualmente no esta de más aclarar el uso que se le da a esa IP. Esa IP generalmente se usa para referenciar a todas las IP’s, en este caso, se permite la conexión a la base de datos desde todas las IP’s posibles. Pienso que en la clase se evito exponer la IP publica de las oficionas/estudio de Platzi.

## Workload

Documento con las generalidades del proyecto

- Cuando borramos una coleccion así, tambien se borran sus esquemas:

db.users.drop()

## Validando datos (SCHEMAS)

Podemos empezar a tener estructuras de documentos que validen campos, esto usando el metodo **createCollection()**, ingresandole el nombre de la colección de la siguiente forma:
```
db.createCollection('users', {
    validator: {
        $jsonSchema: {
            bsonType: 'object',
            properties: {
                name: {
                    bsonType: 'string'
                },
                email: {
                    bsonType: 'string'
                }
            }
        }
    }
})
```

- Si hacemos un insert con campos que no están en el schema de validación Mongo los deja ingresar sin problema, a menos que le agregue:
```
additionalProperties: False
```

- Puedo ver la informacion de los esquemas creados con:
```
db.getCollectionInfos()
```
### Modificaciones en Schemas
Para hacer modificaciones en Schemas, lo hacemos usando:
```
db.runCommand(
    {
        collMod: 'users', 
        validator: {
            new validations...
        }
    })
```
- Nos puede salir el error de que no tenemos poermisos para hacer un collMod:
    1. Vamos a la pagina de Mongodb Atlas.
    2. Entramos al menu Database Access
    3. Le damos editar
    4. Donde dice Built-in Role le damos Atlas admin

## Relaciones

Mongo tiene dos tipos de relaciones entre sus documentos, de forma embebida o referenciada.

- Embeber; dentro del mismo documento
- Referenciar; usando el ObjectId de otra colección
Un método para identificar cuál opción usar nos hacemos la siguiente pregunta:

- ¿Qué tan frecuente es CONSULTADA la información?
- ¿Qué tan frecuente es ACTUALIZADA la información?
- La información se consulta en CONJUNTO o POR PARTES?
- Si se consulta en conjunto se recomienda EMBEBIDA
- Si se consulta por partes se recomienda REFERENCIADA

- En el 90% de los casos, cuando hay una relacion 1:1, esta suele estar EMBEBIDA.

- PERO: Uno de los casos donde se usa la relación REFERENCIADA, es cuando el documento ya es muy grande, cerca del límite de 16 mb y mejor se saca información para mejorar el rendimiento.

- También se pueden usar combinados, si usas el ObjectId, para referenciar el total de datos y agregar los 2 o 3 campos de los más comunes que se usan juntas para evitar la consulta a la 2da colección.

## 1 a muchos

- Embeber REVIEWS o COMENTARIOS de cierto producto es un ejemplo de dónde NO usar este método, porque puede crecer demasiado y reducir el rendimiento de la consulta o exceder los 16 mb de limite.
- Se debe tener cuidado, cuando sean mucho es mejor no embeberlos para que no sea muy pesada la consulta o se superen los 16 MB

## Muchos a muchos
- Estas solo se pueden hacer de forma referenciada

## Desnormalizacion
Es una técnica utilizada para almacenar datos redundantes en un documento para evitar tener que realizar múltiples consultas

## Computer Pattern

- Es un modelo utilizado donde cada vez que ingresamos un nuevo registro, se vayan haciendo precalculos y así cuando se vaya a leer la info, ya estén ciertos calculos hechos. Se suele usar $inc