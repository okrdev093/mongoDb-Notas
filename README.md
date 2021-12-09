# Mongo DB

MongoDB es un gestor de bases de datos no relacional, no utilizaremos tablas o registros como en SQL, sino que haremos usos de colecciones y documentos.

#### Colecciones y Documentos
Podemos ver a las `colecciones` como tablas de SQL y a los `documentos` como los registros (filas de la tabla)  , una colección nos permitirá almacenar  diferentes entidades ( documentos).

En `mongoDB` los `documentos` se representan mediante un formato `JSON`


### Comandos Básicos

##### Mostrar Bases de Datos

    show databases;
    
##### Eliiminar Base de datos

	drop <db_name>
	
##### Utilizar bases de datos 

    use <database_name>; 
      
Si la base de datos no existe el `use` la crea

##### Mostrar Colecciones

    show collections;


#####  Crear  Documentos
Los documentos tienen la sintaxis de objetos JSON
```javascript
user1 = {
"username": 'usuario 1'.
"age" : 23,
"email": 'user@user.com'
}
```

#####  Crear  Colecciones e insertando documentos
la forma mas sencilla de crear una colección es simplemente insertar un documento en ella

```javascript
    db.usuarios.insertOne(user1)
	// o sino 
    db.usuarios.insertOne({"username": 'usuario 1'.
						   "age" : 23,
					       "email":'user@user.com'})
```
Se valida que exista la base de datos  y posteriormente se valida que la colección exista, si estas no existen las crea

##### Ver documentos en una colección

    db.usuarios.find()

retorna un cursor con los documentos almacenados en la colección

  
```javascript
[
  { _id: ObjectId("61ae400db6b67fcf8123034b"), nombre: 'oscar' },
  { _id: ObjectId("61ae420bb6b67fcf8123034c"), nombre: 'cheem' }
]
```

Como podemos ver nos retorna los documentos que insertamos anteriormente y también podemos notar que automáticamente asigna el atributo `_id` a cada documento

Esta cadena esta compuesta por  4 secciones 
```
ObjectId -> 4
1. (TimeStamp)
2. (Identificador para el servidor)
3. (PID)
4. (AutoIncrement)
```

Lo que la hace irrepetible
##### Insertando varios documentos insertMany()
 
 Como vimos anteriormente podemos insertar un documento usando `insertOne` pero aparte  tenemos la función `insertMany()` que nos permite insertar varios documentos, esto lo debemos hacer pasando un array con los elementos que queramos insertar


```javascript
db.usuarios.insertMany([{"username": 'usuario 21',
					   "age" : 24,
				       "email":'user2@user.com',
				       "status":"active"},
				       
				       {"username": 'usuario 31',
					   "age" : 29,
				       "email":'user3@user.com',
				       "status":"inactive"},
				       
				       {"username": 'usuario 41',
					   "age" : 32,
				       "email":'user4@user.com',
				       "status":"active"},
				       
				       {"username": 'usuario 51',
					   "age" : 31,
				       "email":'user5@user.com',
				       "status":"inactive"}])

//nos devolvera 
 insertedIds: {
    '0': ObjectId("61ae47a0b6b67fcf8123034d"),
    '1': ObjectId("61ae47a0b6b67fcf8123034e"),
    '2': ObjectId("61ae47a0b6b67fcf8123034f"),
    '3': ObjectId("61ae47a0b6b67fcf81230350")
  }

```
##### Insertando varios documentos con save() 

Otra manera de insertar documentos es utilizando el método `save()` con save podemos insertar un elemento que no exista en la bd, si el mismo ya existía no se vuelve a insertar pero se actualizan sus campos

```javascript
db.usuarios.save({"username": 'usuario 2',
					   "age" : 24,
				       "email":'user23@user.com'})
```

**

> Mongodb Recomienda Utilizar updateOne o updateMany en vez de save() 

**

### Consultas Básicas

##### Obtener Usuarios Con username 'usuario 4'
```javascript
db.usuarios.find({
	username:'usuario 4'} )

//nos devuelve
[
  {
    _id: ObjectId("61ae47a0b6b67fcf8123034f"),
    username: 'usuario 4',
    age: 32,
    email: 'user4@user.com'
  }
]
// Si sabemos de antemano que vamos a recibir un solo elemento o si asi lo deseamos podemos usar el metodo findOne()

db.usuarios.find({nombre:"cheem"})
>>> [ { _id: ObjectId("61ae420bb6b67fcf8123034c"), nombre: 'cheem' } ]

db.usuarios.findOne({nombre:"cheem"})
>>> { _id: ObjectId("61ae420bb6b67fcf8123034c"), nombre: 'cheem' }

//find() retorna un cursor
//findOne() retorna un documento
```
##### Obtener todos los usuarios con una edad mayor a 18
Para poner una condicion logica a nuestra busqueda debemos pasar otro objeto dentro del campo por el cual queremos buscar en este caso `age` a su vez estamos indicando con `$gt` que vamos a buscar edades mayores a 29.  `age:{$gt:29}` significa `age>29`. `gt: greater than`

```javascript
db.usuarios.find({
	age:{$gt: 29}
} )	

```
Los operadores relacionales son 

    gt: mayor que (>)
    gt: mayor igual que (>=)
    lt: menor que (<)
    lte:menor igual que (=<)
    eq: igual que (=)
    ne: no es igual que (!=)

##### Contar cantidad de registros obtenidos en una consulta

Podemos usar el método `.count()` para obtener  el numero de registro que arroja una consulta encadenándolo al método `find()`

```javascript
db.usuarios.find({
	age:{$lte: 29}
} ).count()	
```

##### Obtener todos los usuarios con una edad mayor a 10 y cuyo status sea activo (varias condiciones)

Si necesitamos hacer una consultar con  varias condiciones usamos el operador `$and:` y pasamos las condiciones como documentos dentro de una lista  `[{campo1:{condicion}, campo2:{condicion} }]`

 
```javascript
db.usuarios.find({
	$and:[
	  {age:{$gt:22}},
	  {status:'active'} 
	]
} )
```
#####  Obtener todos los usuarios que tengan por edad 22,  o 29 o 32 (operador OR)
Al igual que con `$and` debemos pasar una lista con condiciones para evaluar, con la diferencia que con solo una que se cumpla el registro es considerado valido
```javascript
db.usuarios.find({
	$or:[
	  {age:{$eq:22}},
	  {age:{$eq:29}},
	  {age:{$eq:32}} 
	]
} )
```
#####  Obtener todos los usuarios que tengan por edad 22,  o 29 o 32  OTRA FORMA ( $in )

Usando el operador `$in` estamos limitados solo a un campo  donde podemos buscar, pero se optimiza nuestra consulta, pasamos un listado con los valores que estamos buscando dentro de ese campo

```javascript
db.usuarios.find({
	 age:{$in:[22,29,32]}
} )
```


##### Obtener todos los usuarios con atributo status
Nos interesa conocer todos los usuarios que tengan el atributo status independientemente del valor que tenga
para eso usamos el operador `$exists:`

```javascript
db.usuarios.find({
	 status:{$exists: true}
} )
```
#####  Obtener todos los usuarios con status activo
Tenemos 2 opciones para hacer este ejercicio:
La primera como vimos antes es bastante sencilla
```javascript
db.usuarios.find({
	 status:'active'
} )
```

Para la segunda podemos usar el operador `$and` y validar que el atributo `status` exista  y luego pedimos que el status  este activo

```javascript
db.usuarios.find({
	 $and:[
			{status:{$exists: true}},
			{status: 'active'}
	 ]
} )
```
si bien ambas consultas retornan el mismo resultado la segunda condición es mas legible



#####   obtener todos los usuarios con status activo y correo electrónico

```javascript
db.usuarios.find({
	 $and:[
			{status:{$exists: true}},
			{status: 'active'},
			{email:{$exists:true}}
	 ]
} )
```

##### Obtener el usuario con mayor edad
Utilizamos el método `find()` el cual retorna un cursor que ya trae un método `sort()` y dentro de este método describimos como vamos a hacer el orden, con `age: -1` indicamos que el orden va a ser forma descendente por ultimo con el método `limit(1)` le decimos que limite la cantidad de registros a 1

```javascript
db.usuarios.find().sort(
	{
		age: -1	
	}
).limit(1)
```

##### Obtener a los tres usuarios mas jóvenes 
En este caso para ordenar de forma ascendente usamos  `age: 1` pero adicionalmente en el método `.find()` pedimos que el atributo `age` exista

```javascript
db.usuarios.find({
	age:{$exists: true}
	
	}).sort(
	{
		age: 1	
	}
).limit(3)
```

##### Usando expresiones regulares para consultas
Podemos validar los valores de los atributos utilizando expresiones regulares


las expresiones regulares se colocan entre barras `/regExp/` 

```javascript
db.usuarios.find({
	email:/.com$/ 
})

//equivalente a like de mySQL
```
en este caso busca todos los usuarios cuyo correo electrónico finalice con ". com".


```javascript
db.usuarios.find({
	email:/^user/ 
})

//equivalente a like de mySQL Like user1%
```
en este caso busca todos los usuarios cuyo correo electrónico comience con "user" 

```javascript
db.usuarios.find({
	email:/@/ 
})

//equivalente a like de mySQL Like %@%
```
busca los registros cuyo email contenga  "@"

#### Cursores
Un [cursor](https://docs.mongodb.com/manual/reference/method/js-cursor/) es un objeto el cual nos permite conocer todos los documentos obtenidos a partir de una consulta, por default los cursores usan paginación  y cada pagina desplegar un máximo de 20 elementos

##### Métodos de cursores

```javascript
//count()
db.usuarios.find().count()

//devuelve el numero de elementos que se encontraron en una consulta
```

```javascript
//limit()
db.usuarios.find().limit()

//nos limita la cantidad de elementos que queremos obtener
```
```javascript
//skip()
db.usuarios.find().skip(10)
// da un salto y empieza a contar los documentos desde el valor que establescamos, es interesante notar que skip() devuelve otro cursor
```

```javascript
//sort()
db.usuarios.find().sort(
	{age:-1})
// ordena los documentos obtenido en una consulta conforme a los valores de un atributo, tambien retorna otro cursor

```
```javascript
//pretty()
db.usuarios.find().pretty()
//nos formatea la salida de la consulta de tal manera que nos hace mas sencilla la lectura de datos
// 
```


Podemos almacenar a los cursores en variables 
```javascript
var users = db.usuarios.find()
```
la consulta se ejecuta cuando llamamos a la variable

### Proyecciones

Son la forma en las cuales podemos obtener atributos de los documentos de forma muy precisa

```javascript
db.usuarios.find(
	{},// definimos las condiciones
	{   
	    username:true,
		email: true
	} // seleccionamos los atributos que queremos obtener)
```
En este caso obtenemos los atributos que seleccionamos, el atributo `_id` viene seleccionado por defecto, en caso de que no queramos que aparezca en la consulta debemos especifica  `_id:false`

```javascript
db.usuarios.find(
	{
		age:{$gt:24}
	},// definimos las condiciones
	{   
		_id:false,
	    username:true,
		email: true,
		age:true
	} // seleccionamos los atributos que queremos obtener)
```

### Actualizar documentos

```javascript
//obtenemos el documento con nombre 'usuario 51' en una variable
var doc = db.usuarios.findOne({
			username:'usuario 51'
		})

//modificamos la edad
user.age=54
// modificamos el email
user.email='nuevoemail@modificado.com'
// pero para persistir estos cambios en la bd, debemos usar el metodo save() pasando como argumento el documento que queremos actualizar
 db.usuarios.save(doc)
// Esta forma ya no funciona en versiones mas actuales de MongoDb, el metodo save() ya esta deprecado
```
#### Método UpdateOne
Nos permite actualizar un documento primero indicamos la condición en este caso el `_id` y luego en el segundo parámetro  mediante el operador `$set`  definimos los atributos que se van a modificar
```javascript
db.usuarios.updateOne(
    {_id: ObjectId("61ae4ca0b6b67fcf81230354")},
    {
	   $set:{
	    email:'nuevoEmail@modificado.com',
	    age:51
	    }
	}
)
``` 

Si los atributos que intentamos actualizar existen, entonces estos se actualizan, si no existen se van a crear en el documento.
para eliminar uno de los atributos usamos
```javascript
db.usuarios.updateOne(
    {_id: ObjectId("61ae4ca0b6b67fcf81230354")},
    {
	   $unset:{
	    atributo_a_borrar:true}
	}
)
``` 
#### Método updateMany
Permite actualizar varios elementos de una sola vez, en este caso agregamos el campo `bio` a los usuarios que tengan el campo `email`

```javascript
db.usuarios.updateMany(
    {},
    { 
	    $inc:{
		     age:1
	    }
	}
)
``` 

en este otro caso vamos a incrementar +1 la edad de cada usuario
```javascript
db.usuarios.updateMany(
    {email: {$exists: true}},
    {
	   $set:{
	     bio:"agrega tu biografia"
	    }
	}
)
```

### Eliminar  documentos 

#### Método remove()
Recibe un documento con las condiciones  de los elementos que queremos remover

```javascript
// ELIMINA TODOS LOS DOCUMENTOS
db.usuarios.remove({})
// Remueve los usuarios que tiene el  atributo status como 'inactive'

db.usuarios.remove({
	status:'inactive'
})
``` 

Mongo nos recomienda en versiones mas actuales, utilizar `deleteOne` y `deleteMany` funcionan de la misma manera que `updateOne` y `updateMany


#### DeleteOne
#### DeleteMany

#### Eliminar una coleccion

```javascript
//Elimina la tabla
db.usuarios.drop()

//Elimina toda la base de datos

db.dropDatabase()

``` 

### Documentos embebidos

En mongo podemos tener documentos que contengan otros documentos anidados o incluso listas de documentos como vemos en los siguientes ejemplos
```javascript
user13 = {
	"username": "user13",
	"email": "user13@example.com",
	"age": 29,
	"status": "active",
	"address": {
		"zip": 1001,
		"country": "MX"
	},
	"courses": ["Python","MongoDB","Ruby","Java"],
	"comments":[
		{
			body: "Best course",
			like: true,
			tags: ["MongoDB"]
		},
	{
		body: "Super excited",
		like: true
	},
	{
		body: "The course is OK"
	},
	{
		body: "Bad courses, Im disappointed",
		like: false,
		tags: ["bad","course","MongoDB"]
	}

	]
}

user14 = {

	"username": "user14",
	"email": "user14@example.com",
	"age": 29,
	"status": "active",
	"comments": [
		{
			body: "Best course",
			like: true
		}]
}

user15 = {
	"username": "user15",
	"email": "user15@example.com",
	"age": 29,
	"status": "active",
	"comments": [
	{
		body: "Best course",
		like: false
	}]
}

user16 = {
	"username": "user16",
	"email": "user16@example.com",
	"age": 29,
	"status": "active",
	"comments": [
	{
		body: "Best course",
		like: false
	}],
	"address": {
		"zip": 1001,
		"country": "MX",
		"location": {
			"lat": 109,
			"long": -150
		}
}
	}

db.users.insertMany([user13, user14, user15, user16]);
``` 

//TODO
####  Obtener todos los usuarios que radiquen en México.
Para esto debemos hacer es hacer una consulta al atributo `address` el cual es un atributo que contiene un documento embebido, el cual contiene los atributos `zip y country`

```javascript
db.users.find(
{'address.country':'MX'}
)
```
Indicamos a mongo la ruta de  y obtenemos los documentos los cuales cumplan con la condición de que el atributo `address` sea `'MX'`

Ahora buscamos por la latitud
```javascript
db.users.find({
	'address.location.lat': 109 
})
```
Esta notacion de atributos anidados se la conoce como *DOT NOTATION*


#### Actualizar el código postal.

```javascript
db.users.updateMany({
	'address.zip':{$exists:true}
},
{
	$set:{
		'address.zip':110
	}
})
```
#### Añadir dirección a todos los usuarios que no la poseean

```javascript
db.users.updateMany({
	'address':{$exists:false}
},
{
	$set:{
		'address':{
			country:'',
			zip:0000
		}
	}
})

db.users.updateOne(
	{username: "user13"},
	{
	$set:{
		"address.location": {
				lat: -180,
				long: 250 }
		}
});
```
##### Obtener todos los usuarios que tengan en su listado de cursos Python.

```javascript
db.users.find(
	{
		courses: { $exists: true },
		courses: "Python"
	}
);
```
####  Obtener todos los usuarios con por lo menos un comentario positivo.
En este caso para que podamos conocer si el comentario de un usuario fue positivo o negativo, debemos hacer uso del atributo `like`, si es **true** decimos que el comentario fue positivo, si es **false** podemos decir que el comentario fue negativo.
Como ahora vamos a filtrar sobre documentos que se encuentran dentro de una lista, vamos a utilizar `$elemMatch` el cual permite filtrar sobre atributos de documentos dentro de listas

```javascript
db.users.find(
	{
		comments: {
			$elemMatch: {
				like: true}
		}
	}
);
```

```javascript
db.users.find(
	{
	  comments:{
		   $elemMatch:{
			   $and:[
				{like: true},
				{tags:{ $exists:true }} 
				]
		}
	}
	})
```
  
####  Añadir un nuevo comentario positivo al listado de comentarios para el usuario 11.

```javascript
db.users.updateOne(
    {username:'user13'},
    {
	   $push:{
			comments:{
				body:'El curso esta genial',
				like:true}
	   }
	}
)
``` 

```javascript
db.users.updateOne(
    {username:'user13'},
    {
	   $push:{
			courses:'Rust'
	   }
	}
)
``` 
En este caso no utilizamos `$set` porque lo que estamos intentando hacer es insertar un nuevo elemento a un listado
entonces usamos `$push`, este nos permite insertar nuevos elementos al final de una lista.

#### Añade una nueva etiqueta al comentario 4 del usuario 13.
```javascript
db.users.updateOne(
    {username:'user13'},
    {
	   $push:{
			'comments.3.tags':'Lenguaje'
	   }
	}
)
``` 

con `comments.3.tags`Estariamos agregando una nueva etiqueta al comentario 4 (los índices comienzan en 0)

#### Actualiza el segundo comentario del usuario 13.
```javascript
db.users.updateOne(
    {username:'user13'},
    {
	   $set:{
			'comments.1.body':'Re contento del curso'
	   }
	}
)
``` 
### BackUp de Base De datos

`mongodump` nos ayuda a respaldar nuestra base de datos y `mongorestore` nos ayuda a recuperar una base de datos que antes hayamos  respaldado
```javascript
mongodump --db test --out <backup-path>

mongorestore --db codigofacilito <backup-path>
```
