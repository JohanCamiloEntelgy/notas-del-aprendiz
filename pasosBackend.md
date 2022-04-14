# Pasos Backend

> **nota importante:** para crear resolvers, queries, y mutations me basé en el ejemplo de emmy **Homologaciones**

## i) PRISMA

### En schema.prisma

1. crear el modelo en 

```javascript
model documento {
  id Int @id @default(autoincrement()) 
  modulo paramvalores? @relation(fields: [id_modulo], references: [id])
  id_modulo Int?
  nombre_documento String
  descripcion String
}
```
> - para relacionar con modulo se relaciona con el modelo **paramvalores** el campo id_modulo con id <br>
> - esto simula una relación con llave foranea

2. asociar el modelo creado en el modelo **paramvalores** (lin 181)

```javascript
model paramvalores{
   id Int @id @default(autoincrement())
   paramCampo paramcampo? @relation(fields: [idcampo], references: [id])
   idcampo Int?
   valor String
   effdt DateTime
   estado Int
   descr String
   fechacrea DateTime @default(now())
   fechamod DateTime @default(now())
   usuario Int
   plantillacampos plantillacampos[]
   fdtcampos flujo_trabajo[]
   roles nodos_rol[]
   usuarios modulos_usuario[]
   calendarios calendario[]
   tipos_usuario usuario[]
   reportes reporte[]
   documentos documento[] // aquí asocio modelo con variable
}
```
#### RUN
```
npx prisma generate
```

3. crear resolver en src/resolvers/**Documento.js** para definir la relación con modulo
```javascript
function modulo(parent, args, context) {
    return context.prisma.documento.findUnique({ where: { id: parent.id } }).modulo()
}
module.exports = {    
    modulo,
}
```
> - aquí **modulo** corresponde con la variable **modulo** creada en la relación en el schema.prisma de model documento en el paso 1.
> - para esto me basé en src/resolvers/**Homologacioncampo.js** de emmy

### En index.js
4. importar el resolver **documento** y ponerlo en el array de resolvers
```javascript
const documento = require('./resolvers/Documento');


const resolvers = {
  Query,
  Mutation,
  usuario,
  empresa,
  funcion,
  plantilla,
  rol,
  paramvalores,
  paramcampo,
  irolloadercrear,
  proceso,
  plantillacampos,
  pais,
  flujo_trabajo,
  flujotrabajo_accion,
  homologaciones,
  homologacioncampos,
  calendario,
  paramcal,
  reporte,
  param_reporte,
  documento,
  DateTime: GraphQLDateTime
}

```

## ii) GRAPHQL

### En schema.graphql

5. crear el **type documento {}** que es como el modelo en graphql
```javascript
type documento {
  id: ID!  
  modulo: paramvalores
  nombre_documento: String
  descripcion: String
}
```

6. crear las querys dentro de **type Query {}**
```javascript
consultarDocumentos: [documento]
getDocumento(id: Int!): documento!
```

7. crear las mutations dentro de **type Mutation {}**
```javascript
createDocumento(nombre_documento: String!, descripcion: String!, id_modulo: Int!): documento
updateDocumento(id:Int!, nombre_documento: String!, descripcion: String!, id_modulo: Int!): documento  
deleteDocumento(id:Int!): documento
```


## iii) Querys

8. Crear query src/resolvers/Querys/**documento.js** para implementar las Querys declaradas de **schema.graphql**
```javascript
//===========================READ DOCUMENTO======================//
function consultarDocumentos(parent, args, context) {
	const { userId } = context;
	if (userId != null) {
		return context.prisma.documento.findMany();
	}
}

function getDocumento(parent, args, context) {
	const { userId } = context;
	if (userId != null) {
		return context.prisma.documento.findUnique({
			where: {
				id: args.id
			}
		})
	}
}
module.exports = {
	consultarDocumentos,
	getDocumento
}
```
> basada en query **homologaciones.js**

### Query.js
9. hacer require de mi query creado **documento** y ponerlo en mixing
```javascript
const documento = require('./Querys/documento')

//Add separate mutation files here for combine static export with dynamic form
const mixing = {
  ...FlujoTrabajo,
  ...homologaciones,
  ...homologacioncampos,
  ...documento
}
```
#### RUN
```
node src/index
```
> cada vez que se hagan cambios en **schema.graphql** o en otro lado del back en general levantar de nuevo el back si no se tiene instalado **nodemon**

10. testear las querys creadas en **apollo server**

```javascript
query { consultarDocumentos{ id, descripcion, modulo {
  id, descr
} } }
```

```javascript
query {
  getDocumento(id: 1) {
    nombre_documento,
    descripcion
  }
}
```


## iv) Mutations

11. Crear mutation para CUD src/resolvers/Mutations/**documento.js** para implementar las Mutations declaradas de **schema.graphql**

```javascript
async function createDocumento(parent, args, context, info){
		
	return await context.prisma.documento.create({
			data: {
                id_modulo: args.id_modulo,
                nombre_documento: args.nombre_documento,
				descripcion: args.descripcion,                
			}
	});
}
async function updateDocumento(parent, args, context, info){
	return await context.prisma.documento.update({
        where: { id: args.id },
        data: {
            id_modulo: args.id_modulo,
            nombre_documento: args.nombre_documento,
            descripcion: args.descripcion,
        }
      })
}
async function deleteDocumento(parent, args, context, info){
	return await context.prisma.documento.delete({
    where: {
      id: args.id
    }
  })
}

module.exports = {
    createDocumento,
    updateDocumento,
    deleteDocumento
}
```
> basada en mutation **homologaciones.js**

12. testear las mutations creadas en **apollo server**
```javascript
mutation {
  createDocumento(nombre_documento: "doc_ejemplo", descripcion: "descripcion ejemplo", id_modulo: 1) {
    id, nombre_documento
  }
}
```
```javascript
mutation {
  updateDocumento(id: 1, nombre_documento: "editado doc", descripcion: "desc editada", id_modulo: 1) {
    id, nombre_documento, descripcion
  }
}
```
```javascript
```
