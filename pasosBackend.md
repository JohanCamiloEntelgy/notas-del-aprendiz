# Pasos Backend

> **nota importante:** para crear resolvers, queries, y mutations me basé en el ejemplo de emmy **Homologaciones**

## En schema.prisma

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
> para relacionar con modulo se relaciona con el modelo **paramvalores** el campo id_modulo con id

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

## En schema.graphql

3. crear el **type documento {}** que es como el modelo en graphql
```javascript
type documento {
  id: ID!  
  modulo: paramvalores
  nombre_documento: String
  descripcion: String
}
```

4. crear las querys dentro de **type Query {}**
```javascript
consultarDocumentos: [documento]
getDocumento(id: Int!): documento!
```

5. crear las mutations dentro de **type Mutation {}**
```javascript
createDocumento(nombre_documento: String!, descripcion: String!, id_modulo: Int!): documento
updateDocumento(id:Int!, nombre_documento: String!, descripcion: String!, id_modulo: Int!): documento  
deleteDocumento(id:Int!): documento
```
6. crear resolver en src/resolvers/**Documento.js** para definir la relación con modulo
```javascript
function modulo(parent, args, context) {
    return context.prisma.documento.findUnique({ where: { id: parent.id } }).modulo()
}
module.exports = {    
    modulo,
}
```
> para esto me basé en src/resolvers/**Homologacioncampo.js** de emmy

```javascript
```
