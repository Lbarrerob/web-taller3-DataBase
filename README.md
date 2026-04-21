# Respuestas a las preguntas del Task

## Question 1
MongoDB si lo considera duplicado, ya que para la creación en la base de datos primero hace una comparación del nombre pasando todo a `lowercase` lo que hace que sí sean iguales.

La línea en `cars.service.ts` está en la función de **create**   
`createCarDto.nombre = createCarDto.nombre.toLowerCase();`  
y lo que hace es cambiar a minuscula el atributo que llega para asignar a nombre del carro, para luego poder hacer la condicion que aplica el metodo de create de que si ya existe en la base de datos mande error.

Esa línea es la que permite que la restricción `unique` se pueda cumplir en términos de caracteres y no se base en la diferencia de mayúsculas y minúsculas, ya que hace que pueda comparar dos nombres para determinar si son iguales o no especificamente por el orden de las letras que contiene.

## Question 2
Ambas validaciones existen porque ocurren en momentos diferentes y son accedidas en capas distintas. `findOne` valida dentro del servicio para evitar que Mongoose lance CastError con un HTTP 400 en vez de 500. Mientras que en `remove`el validador evita que el controlador reciba un ID inválido y responde con HTTP 400 antes de entrar al servicio.

La diferencia entre las respuestas de HTTP es que cuando hay una respuesta de 400 refiere a un error fue causado por el cliente que envía información inválidos o incompletos. Y la respuesta de 500 es por errores que ocurrió dentro del servidor, el cliente pudo haber enviado algo válido o inválido, pero el problema fue interno en le backend.

## Question 3
El método **create** necesita `try/catch` porque este método busca guardar nueva información en la base de datos y ese bloque permite evitar errores como guardar información duplicada o inconsistente. Mientras que **findAll** no necesita este bloque ya que es un método que es sólo de consulta y de lectura, no busca hacer ningún tipo de modificación que pueda causar un error o inconsistencia, por lo que no es necesario que maneje errores.

Si se quita el `try/catch` del método de **create**, MongoDB manda error 11000 y el mensaje que se mandaría para el cliente sería de HTTP 500 ya que ese representa un error interno en la base de datos.

## Question 4
En el metodo de **update** se ralizan dos peticiones, la primera es para buscar el documento que se quiere actualizar y la segunda es para actualizarlo. Esto puede trar problemas con la respuesta de api ya que, al no consultar de nuevo el documento cambiado, se pueden enviar respuestas incosistentes luego de haber hecho un cambio.

Un escenario en el que esto podría suceder es si se manda actualizar el nombre de un carro, pero la respuesta de la api envia el mismo carro consultado, entonces se mostraría el carro con el nombre sin actualización realizada.

## Question 5
Al usar `forRoot(process.env.MONGODB_URL...)` se evaluaría inmediatamente aún si las variables de entorno no han sido cargadas, por lo que su valor pasaría a ser undefined y se usaría la url por defecto. 

Por otra parte, `forRootAsync` con **useFactory** espera a que estas variables sean cargadas antes de crear la conexión.

## Question 6
Si el estudiante olvida importar CarsModule en AppModule la aplicación sí iniciaría, pero Nest nunca registraría las rutas de `CarsModule`, entonces cuando alguien intente acceder a `/cars` recibirá un mensaje HTTP 404.

Si sí importa `CarsModule`, pero olvida `MongooseModule.forFeature`, Nest no podría inyectar el modelo de **Car** y la aplicación fallaría al iniciarse. El archivo que se podría revisar para diagnosticar puede ser el de `app.module.ts`.

## Question 7
La ventaja de que `deleteOne` no haga `findOne(id)` antes de eliminar el registro es evitar hacer una doble consulta al mismo registro. 

`deletedCount` puede ser 0 bajo las condiciones de que el ObjectId exista como formato, pero no exista ningún otro documento con ese id en MongoDB.

## Question 8
Mover la validación del pipe al servicio hace que los datos inválidos lleguen más lejos dentro de la aplicación y obliga a repetir validaciones en varios lugares. Si se elimina `@Injectable()`, se podría intanciar el pipe, sin embargo, no es una buena practica.

## Question 9
Para el primer caso, la nueva configuración no afecta la ejecución ni el comportamiendo de la app ya que son configuraciones independientes. Pero en el segundo caso, mover `app.enableCors()` después de `app.listen()` sí causaría problemas por lo que CORS debe configurarse antes de que el servidor empiece a recibir peticiones.