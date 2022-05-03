# Curso-TypeScript-Tipos-Avanzados-Funciones
![image](https://user-images.githubusercontent.com/42653934/166395339-72fdbbeb-dc25-446f-bb2d-9c1a3b8fcec5.png)

## Configuración del proyecto con ts-node

Dentro de nuestro proyecto, vamos a hacer el `git init` y generamos el gitignore con [**gitignore.io](http://gitignore.io)** para windows, linux, mac y node.js.

También hacemos el `npm init -y`, instalamos typescript en el proyecto con `npm i typescript -D` y podemos verificar que este completamente instalado con `npx tsc —version`.

Luego vamos a hacer el `npx tsc --init` para obtener nuestro archivo tsconfig.json.

Igualmente configuramos nuestro .editorconfig de la siguiente manera para tener las mismas configuraciones que el resto del curso. Es necesario tener la extensión de editor config para que las especificaciones del archivo sean tomadas por el editor de código.

```xml
# Editor configuration, see https://editorconfig.org
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

[*.ts]
quote_type = single

[*.md]
max_line_length = off
trim_trailing_whitespace = false>
```

Con la librería [**ts-node**](https://typestrong.org/ts-node/) podemos correr typescript en node sin tener que transpilarlo. Para hacerlo vamos a ejecutar: 

```bash
npm install -D ts-node
```

Luego para correr nuestro código, hacemos: 

```bash
#npx ts-node ruta/archivo.ts
npx ts-node src/demo.ts
```

<aside>
💡 Si bien ts-node nos va a permitir ejecutar typescript en node, esto es en desarrollo, para producción vamos a tener que traspilarlo igualmente.

</aside>

## Enums

Los enums nos permiten configurar un set de opciones, son parecidos a los literal types pero su uso es distinto.

Lo usamos con la palabra reservada `enum` y denominamos nuestro enum con el nombre en mayúsculas. Dentro de los enums, las keys las solemos escribir en mayúsculas, y los values pueden ser de tipo number o string

```tsx
enum ROLES {
  ADMIN = "admin",
  SELLER = "seller",
  CUSTOMER = "customer"
}

type User = {
  username: string;
  role: ROLES;
}

const francoUser: User = {username: "Franco", role: ROLES.ADMIN}
```

Las ventajas son que yo puedo ver las opciones de lo que voy a asignar de forma directa a nuestras propiedades, y a su vez, en caso de que la propiedad de nuestro enum cambie en un futuro, también lo hará en su implementación.

---

Vamos a analizar una librería para hacer aplicaciones multiplataforma llamada [**capacitor](https://capacitorjs.com/)** para ver cómo hacen uso de los enums y como estos nos ayudan.

Si bien no lo vamos a poder ver en funcionamiento como tal, ya que lo que hace este plugin de capacitor es levantar una cámara, nos va a servir para ver como se utilizan los enums en el mismo. Lo instalamos con:

```tsx
npm install @capacitor/camera
```

Con el ejemplo de la documentación vamos a poder analizar el poder de los enums.

```tsx
import { Camera, CameraResultType } from '@capacitor/camera';

const takePicture = async () => {
  const image = await Camera.getPhoto({
    quality: 90,
    allowEditing: true,
    resultType: CameraResultType.Uri
  });

  // image.webPath will contain a path that can be set as an image src.
  // You can access the original file using image.path, which can be
  // passed to the Filesystem API to read the raw data of the image,
  // if desired (or pass resultType: CameraResultType.Base64 to getPhoto)
  var imageUrl = image.webPath;

  // Can be set to the src of an image now
  imageElement.src = imageUrl;
};
```

Al explorar las propiedades, podemos ver las opciones predefinidas en para que no exista la posibilidad de equivocarnos y pasar un argumento inválido.

## Tuplas

Nos permiten definir un array fuertemente tipado tanto en posición como en el “límite de elementos”, esto por defecto no esta en javascript ([**Esta propuesto en los ES Proposals en Stage 2**](https://www.proposals.es/proposals/Record%20%26%20Tuple)).

```tsx
const prices: (number | string)[] = [1,3,2,1,2,'as']
prices.push(2)
prices.push('Hello')

let user:[string,number,boolean] = ['Franco', 25,true];

//user = [36,'Franco',24]
//user = []
user = ['Fran', 25, false]
```

Con las tuplas además podemos hacer desestructuración: 

```tsx
const [username, age] = user; 
console.log(username); //'Franco'
```

Este tipo de desestructuración las podemos ver en el react hook de `useState`. 

<aside>
❗ Las tuplas de Typescript solo entran en acción en casos de reasignación de variables, pero igualmente nos siguen permitiendo hacer `.push()` que en el caso de la proposal no, ya que no van a tener métodos que muten directamente el array.

</aside>

**No debemos confiarnos del lÍmite en longitud, ya que este puede cambiar mutando el array.** Lo único que debemos “respetar” para poder hacerlo, es utilizar los mismos tipos que especificamos en la tupla.

```tsx
let user:[string,number] = ['Franco', 25];
console.log(user) // ['Franco', 25]

user.push('Carrara');
//user.push(true); DA ERROR PORQUE NO ESTA BOOLEAN ESPECIFICADO EN LA TUPLA
console.log(user) // ['Franco', 25, 'Carrara']
//console.log(user[2]) ESTO DA ERROR
console.log(user.at(-1)) // 'Carrara'

user.map(item => console.log(item))
// 'Franco'
// 25
// 'Carrara'
```

Con las tuplas, también podemos pasar un numero indefinido de valores de un tipo: 

```tsx
const args: [boolean, ...number[], string] = [true, 2, 3, 4, 5, 6, "seven"]
console.log(args)

//[ true, 2, 3, 4, 5, 6, "seven"]
```

También podemos pasar valores opcionales

```tsx
let newTuple2: [boolean, string, number?] = [true, "variable"]; //Number is optional
console.log(newTuple2);

//[ true, "variable" ]
```

<aside>
💡 Créditos a Victor por los dos últimos ejemplos sacados de su [**comentario**](https://platzi.com/comentario/3540961/).

</aside>

## Unknown types

Es un tipo que nos dice que la variable es desconocida, es similar a `any`, pero sin quitar el análisis de código estático que nos brinda TypeScript.

```tsx
let anyVar: any;
anyVar = true;
anyVar = 'Franco'
anyVar = undefined; 

let isNew: boolean = anyVar; //Esto funciona porque anyVar, al tener any podría ser un boolean.

let unknownVar: unknown;
unknownVar = true;
unknownVar = "Franco";
unknownVar = 25;
unknownVar = {};

//unknownVar.toUpperCase(); Aca marca error porque no sabe si va a ser un string.

if (typeof unknownVar === "string"){
  unknownVar.toUpperCase()
}
```

También la podemos usar en funciones si no sabemos exactamente que nos va a devolver.

```tsx
const parse = (str: string): unknown => {
	return JSON.parse(str)
}
```

Para realizar el ejemplo de la reasignación, lo podemos hacer de la siguiente manera: 

```tsx
let unknownVar: unknown;
unknownVar = true;
unknownVar = "Franco";
unknownVar = 25;
unknownVar = {};

//let isNew: boolean = unknownVar; Nos da error ya que pide que verifiquemos los tipos.
let isNew: boolean;

if (typeof unknownVar === "boolean"){
  isNew = unknownVar
}
```

TypeScript, en este caso, a diferencia de como pasa con `any`, nos marca error si no verificamos los tipos antes.

## Never type

Se usa para funciones que nunca van a terminar o que detienen el programa. 

```tsx
const esUnCicloSinFin = () => {
  while(true){
    console.log('El Rey León')
  }
}
```

En este código, TS infiere que el tipo es never, ya que su ejecución será infinita.

---

En este caso, al hacer un throw se detiene la ejecución, por lo que TypeScript infiere que es never.

```tsx
const fail = (message: string) => {
  throw new Error(message)
}

const example = (input:unknown) => {
  if(typeof input === 'string'){
    return 'Es un string'
  }
  else if (Array.isArray(input)){
    return 'Es un array'
  } else {
    return fail('Not Match')
  }
}
console.log(example('Franco')) //'Es un string'
console.log(example(['Franco','Hola'])) // 'Es un array'
console.log(example({username: 'Franco'})) // error: Uncaught Error: Not Match
console.log(example('Franco de nuevo')) // NUNCA SE EJECUTA

```

---

En el siguiente caso si bien es infinito, TS infiere que es void: 

```tsx
const badFor = () => {
  for(let i = 1; i < 10; i){
    console.log(i)
  }
}
```

Lo mismo pasa con este ejemplo de función recursiva sin un escape. Si bien la función va a ser infinita (Y no es difícil de notarlo por nosotros mismos), TS no detecta el tipo `Never` y lo infiere como void.

```tsx
const badRecursion = () => {
  if(true){
    console.log('Oh sh*t here we go again')
    badRecursion()
  }
}
```

## Parámetros opcionales y nullish-coalescing

Los parámetros opcionales deben encontrarse al final.

```tsx
export const createProduct = (
  id: string | number,
	isNew: boolean,
  stock?: number,
) => {
  return {
    id,
    stock,
    isNew
  }
}
const p1 = createProduct('Remera Fachera facherita', true, 20);
console.log(p1)//{ id: "Remera Fachera facherita", stock: 20, isNew: true }
const p2 = createProduct('Gorrita', false);
console.log(p2) //{ id: "Gorrita", stock: undefined, isNew: false }
```

Con el `logical OR operator (||)` podemos cambiar ese undefined por un valor arbitrario, pero lo que evalúa es cualquier valor `falsy`. 

```tsx
export const createProduct = (
  id: string | number,
	isNew: boolean,
  stock?: number,
) => {
  return {
    id,
    stock: stock || 10,
    isNew
  }
}

const p2 = createProduct('Gorrita', false, 0);
console.log(p2) //{ id: "Gorrita", stock: 10, isNew: false }
```

Esto lo podemos corregir utilizando el `Nullish Coalescing (??)` que nos va a evaluar si el parámetro es null o undefined.  

```tsx
export const createProduct = (
  id: string | number,
	isNew: boolean,
  stock?: number,
) => {
  return {
    id,
    stock: stock ?? 10,
    isNew
  }
}

const p2 = createProduct('Gorrita', false, 0);
console.log(p2) //{ id: "Gorrita", stock: 0, isNew: false }
```

**Documentación:** 

[Logical OR (||) - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR)

[Nullish coalescing operator (??) - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator)

## Parámetros por defecto

Además del Nullish coalescing, podemos pasar parámetros por defecto con los **Default function parameters** de Javascript y lo hacemos en los parámetros con un signo igual. 

```tsx
export const createProduct = (
  id: string | number,
	isNew: boolean,
  stock: number = 0,
) => {
  return {
    id,
    stock,
    isNew
  }
}

const p2 = createProduct('Gorrita', false, 0);
console.log(p2) //{ id: "Gorrita", stock: 0, isNew: false }
```

## Parámetros rest

Se apoyan en la flexibilidad de JS de poder enviar parámetros con total flexibilidad y poder pasar la cantidad de parámetros que queramos. 

```tsx
const sum = (...arg:number[]) => {
  const addition = arg.reduce((acc,num)=> acc + num, 0)
  return addition;
}
const value = sum(1,2,3,4)
console.log(value) //10
```
## ****Sobrecarga de funciones****

En funciones que puedan retornar más de un tipo, tenemos que hacer una verificación de tipos posterior para poder utilizarlas por más que sepamos el tipo retornado o a tratar, ya que TS no puede inferirlo por si mismo. Esta sobrecarga de funciones, solo funciona en las funciones clásicas y no en las arrow functions.

```tsx
// 'Fran' => ['F', 'r', 'a', 'n'] : string => string[]
// ['F', 'r', 'a', 'n'] => 'Fran' : string[] => string

function parseStr(input: string | string[]): string | string[]{
  if(Array.isArray(input)){
    return input.join('');
  } else {
    return input.split('')
  }
}
const rta1 = parseStr('Franco')
//rta1.reverse(); Marca error, ya que TS no sabe que es un array. Deberiamos hacer una verifiación de tipos.
console.log('rta1', rta1)
const rta2 = parseStr(['F', 'r', 'a', 'n'])
console.log('rta2', rta2)
```

Esto lo resolvemos de la siguiente manera: 

```tsx
// 'Fran' => ['F', 'r', 'a', 'n'] : string => string[]
// ['F', 'r', 'a', 'n'] => 'Fran' : string[] => string

function parseStr(input: string): string[];
function parseStr(input: string[]): string;

function parseStr(input: string | string[]): string | string[]{
  if(Array.isArray(input)){
    return input.join('');
  } else {
    return input.split('')
  }
}
const rta1 = parseStr('Franco')
console.log('rta1', rta1)

const rta2 = parseStr(['F', 'r', 'a', 'n'])
console.log('rta2', rta2)
```

Si bien la lógica interna no cambia, no vamos a tener que hacer una verificación de tipos posterior ya que TS va a poder inferir qué tipo de dato es el output de una función. 

**Como evitar malas prácticas:** 

- En caso de en nuestra sobrecarga algún caso con `unknown`o `any`, debemos dejarlo al final, ya que sino va a fallar la verificación de tipos.
- En caso de retornar lo mismo, y estamos realizando la sobrecarga de funciones debido a los parámetros, capaz sea mejor idea utilizar los optional params.
- En los casos en los que los tipos de entrada sean de distintos tipos, pero el output sea del mismo tipo, es mejor utilizar los union types en vez de la sobrecarga de funciones.

## Interfaces

Sirven para lo mismo que los tipos pero con algunas diferencias.

Con tipos seria de la siguiente manera: 

```tsx
type Sizes = 'S' | 'M' | 'L' | 'XL'
type Product = {
  id: string | number;
  title: string;
  stock: number;
  size: Sizes
}

const products: Product[] = [];

products.push(
  {
    id: 1,
    title: 'product 1',
    stock: 10,
    size: 'XL'
  }
)
```

Y con interfaces, sería así:

```tsx
type Sizes = 'S' | 'M' | 'L' | 'XL'
interface Product {
  id: string | number;
  title: string;
  stock: number;
  size: Sizes
}

const products: Product[] = [];

products.push(
  {
    id: 1,
    title: 'product 1',
    stock: 10,
    size: 'XL'
  }
)
```

Si bien son similares, tienen algunas diferencias: 

- Las interfaces se pueden extender de otras interfaces, por lo que puedo heredar los tipos de una interfaz distinta.
- A las interfaces se le pueden agregar campos extra luego de ser creadas.

Extendiendo con literal types:

```tsx
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: boolean 
}

const bear = getBear();
bear.name;
bear.honey;
```

Añadiendo nuevos campos con literal types:

```tsx
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

 // Error: Duplicate identifier 'Window'.
```

Extendiendo con interfaces: 

```tsx
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
```

Añadiendo nuevos campos con Interfaces:

```tsx
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

[Documentation - Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#:~:text=typed%20type%20system.-,Differences%20Between%20Type%20Aliases%20and%20Interfaces,-Type%20aliases%20and)

## Interfaces complejas

Muchas veces las interfaces pueden tener el mismo nombre que clases, por este motivo es recomendable agregar el sufijo Interface a las interfaces.

```tsx
class Person {
    @code...
}

interface PersonInterface {
    @code...
}
```

<aside>
💡 Créditos a Irving Juárez ([**Ver comentario**](https://platzi.com/comentario/3508679/))

</aside>

---

Para manejar los datos, tenemos que dividirlos en distintos archivos para hacer este código más mantenible, separar sus responsabilidades y hacerlo más legible. 

Así como lo vimos en el curso de fundamentos, a los archivos de modelado de datos debemos llamarlos de la siguiente forma: `nombre.model.ts` y a los archivos que van a tratar dichos datos de la siguiente manera: `nombre.service.ts`. A su vez, los archivos que van a hacer uso de estos servicios deben llamarse `service.ts`.

## Extender interfaces

Con la herencia, podemos extender distintos campos mediante la herencia como en la programación orientada a objetos.

Para hacerlo, lo hacemos de la siguiente manera: 
 

```tsx
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
```

Sin la herencia, lo que deberíamos hacer es repetir código en cada modelo:

`category.model.ts`

`products.model.ts`

`user.model.ts`

```tsx
export interface Category {
  id: string |number;
  createdAt: Date;
  updatedAt: Date;
  name: string;
}
```

```tsx
import { Category } from "../category/category.model";

export type Sizes = 'S' | 'M' | 'L' | 'XL'
export interface Product {
  id: string |number;
  createdAt: Date;
  updatedAt: Date;
  title: string;
  stock: number;
  size: Sizes;
  category: Category;
}
```

```tsx
export enum ROLES {
  ADMIN = 'admin',
  SELLER = 'seller',
  CUSTOMER = 'customer'
}

export interface User {
  id: string |number;
  createdAt: Date;
  updatedAt: Date;
  username: string;
  role: ROLES;
}
```

Si vemos bien, en todos los modelos estamos repitiendo los campos de `id`, `createdAt` y `updatedAt` que lo podemos resolver muy fácil extendiendolo de otra interface.

Creamos un archivo `base.model.ts`:

```tsx
export interface BaseModel {
  id: string |number;
  createdAt: Date;
  updatedAt: Date;
}
```

Y ahora simplemente con hacer los imports y extendiendo, ya podemos heredar esas propiedades de nuestra interface base.

`category.model.ts`

`products.model.ts`

`user.model.ts`

```tsx
import { BaseModel } from "../base.model";

export interface Category extends BaseModel{
  name: string;
}
```

```tsx
import { Category } from "../category/category.model";
import { BaseModel } from "../base.model";

export type Sizes = 'S' | 'M' | 'L' | 'XL'
export interface Product extends BaseModel{
  title: string;
  stock: number;
  size: Sizes;
  category: Category;
}
```

```tsx
import { BaseModel } from "../base.model";

export enum ROLES {
  ADMIN = 'admin',
  SELLER = 'seller',
  CUSTOMER = 'customer'
}

export interface User extends BaseModel {
  username: string;
  role: ROLES;
}
```

## Propiedades de solo lectura

En los atributos que no queramos que sean modificadas, podemos aplicar `readonly` que convierte estos atributos a solo lectura por lo que no se va a poder sobrescribir ni modificar. 
En nuestro caso, es muy útil para los campos de `id` y de `createdAt` ya que estos no deben cambiar.

```tsx
export interface BaseModel {
  readonly id: string |number;
  readonly createdAt: Date;
  updatedAt: Date;
}
```

## Ejemplo de CRUD

En esta clase comenzamos a hacer un CRUD para nuestra tienda. CRUD viene de las siglas en ingles de Create Read Update and Delete.

Para llenar los datos de los productos hacemos uso de la librería Faker.js, que la instalamos con `npm install @faker-js/faker --save-dev`.

Vamos a modificar nuestra interface Product para agregar nuevos campos:

```tsx
export interface Product extends BaseModel{
  title: string;
  image: string;
  description: string;
  stock: number;
  size: Sizes;
  color: string;
  price: number;
  category: Category;
  isNew: boolean;
  tags: string[];
}
```

# Utility Types

## DTOs

Son los Data Transfer Objects o Objetos de Transferencia de Datos. Es una utilidad de TS para representar objetos que viajan por internet, ya que en algunos casos, al hacerse el `JSON.stringify()` , ya que lo que mandamos es un string, en algunos casos, al hacer el `JSON.parse()` algunos tipos pueden ser los incorrectos.

[DTO - A typescript utility type](https://dev.to/tamj0rd2/dto-a-typescript-utility-type-4o3m)

## Omit y Pick

Omit es utilizado como su nombre lo indica, para omitir propiedades o campos que seleccionemos. 

En los casos en los que necesitamos extender la interfaz, nos conviene hacerlo con `interface` y agregar las propiedades, en cambio, cuando solo queremos omitir tipos, es mejor hacerlo con `type`. 

```tsx
type DTO1 = Omit<Base, 'omited1' | 'omited2'>

interface DTO2 extends Omit<Base, 'omited1' | 'omited2'>{
  newProp: string;
}
```

```tsx
export interface Product extends BaseModel{
  title: string;
  image: string;
  description: string;
  stock: number;
  size: Sizes;
  color: string;
  price: number;
  category: Category;
  isNew: boolean;
  tags: string[];
}

interface CreateProductDTO2 extends Omit<Product, 'id' | 'createdAt' | 'updatedAt' | 'category'>{
  categoryId: string;
}
```

Pick es lo contrario a Omit. Este Utility Type nos permite utilizar los campos que yo elijo. 

```tsx
type DTO1 = Pick<Base, 'picked1' | 'picked2'>

interface DTO2 extends Pick<Base, 'picked1' | 'picked2'>{
  newProp: string;
}
```

### Buenas prácticas

Una buena práctica es que los DTOs tengan su propio archivo.

### Pequeña aclaración sobre nuestro código en esta clase

En el caso de las categorías en los productos, cuando creamos un producto no es que creamos una categoría a la vez, esta ya viene creada y solo la relacionamos, tenemos que mandar solamente el ID de la categoría a la que está relacionado el producto.

<aside>
💡 Créditos a **Axel Enrique Galeed Gutierrez ([Ver Comentario](https://platzi.com/comentario/3567910/))**

</aside>

## Partial y Required Type

El utility type `partial` nos permite definir que todos los parámetros de una interfaz son opcionales.

```tsx
interface InterfaceName extends Partial<Interface> {
		statements
}

type TypeName = Partial<TypeOrInterface>;
```

`Required` nos permite colocar todos los parámetros como obligatorios.

```tsx
interface InterfaceName extends Required<Interface> {
		statements
}

type TypeName = Required<TypeOrInterface>;
```

### Reutilización de DTOs

Podemos reutilizar nuestro DTO normalmente, ya que son interfaces.

```tsx
import { Product } from "./product.model";

export interface CreateProductDto extends Omit<Product, 'id' | 'createdAt' | 'updatedAt' | 'category'> {
    categoryId: string;
}

export interface UpdateProductDto extends Partial<CreateProductDto> {

}
```

<aside>
💡 Créditos a **Axel Enrique Galeed Gutierrez** ([**Ver Comentario**](https://platzi.com/comentario/3568608/))

</aside>

## Readonly Type

Con este utility type podemos establecer todos los parámetros como solo lectura para evitar que sean cambiados.

```tsx
interface DTO1 extends Readonly<Base> {
		props: string;
}

type DTO2 = Readonly<Base>;
```

### Anidamiento de Utility Types

A su vez podemos anidar distintos tipos utility types para fusionar características de los distintos utility types. 

```tsx
interface DTO1 extends UtilityType1<UtilityType2<Base>> {
		props: string; 
}

type DTO2 = UtilityType1<UtilityType2<Base>>;
```

Ejemplo:

```tsx
export interface FindProductDto extends Readonly<Partial<Product>> {
		props: string;
}

export type FindProductDto2 = Readonly<Partial<Product>>
```

<aside>
💡 Créditos a **Axel Enrique Galeed Gutierrez ([Ver Comentario](https://platzi.com/comentario/3568689/))**

</aside>

## Acceder al tipado por indice

Podemos acceder al tipado de una propiedad de forma similar a como accedemos a los valores dentro de un array, pero aplicado a una interfaz para obtener el tipo de dato de una propiedad. Al colocar el dato de esta forma evitamos que en caso de que cambie algún tipo de dato en nuestra interfaz aparezcan errores en los casos que no hayan inconsistencias propias del código.

```tsx
InterfaceName['property']
```

Ejemplo:

```tsx
const updateProduct = (idSearch: Product['id'])
```

Pasamos como tipo de dato el valor de la propiedad dentro de la interfaz. 

<aside>
💡 Créditos a **Irving Juárez ([Ver Comentario](https://platzi.com/comentario/3509240/))** y **Axel Enrique Galeed Gutierrez ([Ver Comentario](https://platzi.com/comentario/3568737/))**

</aside>

## ReadonlyArray

Este Type previene las mutaciones en los arrays. Este Utility Type quita los métodos mutables como `.pop()`, `.shift()`, `.push()`, `.unshift()` entre otros, por lo que vamos a tener un array de solo lectura.

```tsx
const inmutableArray = ReadonlyArray<Type> = [value];
```

```tsx
function foo(arr: ReadonlyArray<string>) {
  arr.slice(); // okay
  arr.push("hello!"); // error!
}

/** Desde la versión 3.4 de TS tambien podemos usar esta sintaxis **/

function foo2(arr: readonly string[]) {
  arr.slice(); // okay
  arr.push("hello!"); // error!
}
```

**Ejemplo:** 

```tsx
const numberArray = ReadonlyArray<number> = [1,2,3,4,5];
```

---

En los casos en las que tenemos una interfaz que por dentro tiene un array, no vamos a poder reasignar el array, pero si lo vamos a poder mutar. Para corregir esto, dentro de la interfaz, debemos aplicar el `ReadonlyArray`

**Ejemplo:**

```tsx
export interface FindProductDto extends Readonly<Partial<Omit<Product, 'tags'>>>{
	readonly tags: ReadonlyArray<string>;
}
```
