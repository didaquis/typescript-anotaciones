# 📚 TypeScript: breves anotaciones sobre el lenguaje  

Listado personal de anotaciones, trucos, recordatorios, utilidades o ejemplos interesantes para TypeScript.  

![typescriptlang.png](./assets/typescriptlang.png) 

## Tabla de Contenido
- [Diccionarios](#diccionarios)
- [Mapeando opciones ](#mapeando-opciones)
- [Extendiendo la clase Error](#extendiendo-la-clase-error)
- [Utilización de keyof](#utilización-de-keyof)
- [Utilización de Omit](#utilización-de-omit)
- [Obtener los tipos de una librería de terceros no tipada](#obtener-los-tipos-de-una-librería-de-terceros-no-tipada)



----------------------------------------------------------
## Diccionarios

```typescript
type Dictionary = { [key: string]: unknown };

const a: Dictionary = {
  name: 'didi',
  age: 39,
  male: true
}

if (a.age > 25) { // This is not allowed on TypeScript
  console.log('true b')
}

if (Object.hasOwn(a, 'age') && typeof a.age === 'number' && a.age > 25) { // TypeScript force you to check if property exists before using it
  console.log('true a')
}
```
----------------------------------------------------------  
## Mapeando opciones

```typescript
export enum PersistenceStrategy {
	MYSQL = 'mysql',
	MONGODB = 'mongodb',
	POSTGRESQL = 'postgresql',
}

export const getPersistenceStrategy = (): PersistenceStrategy => {
	const usedPersistenceStrategy = process.env.PERSISTENCE_STRATEGY;

	switch (usedPersistenceStrategy) {
		case PersistenceStrategy.MYSQL:
			return PersistenceStrategy.MYSQL;
			break;

		case PersistenceStrategy.MONGODB:
			return PersistenceStrategy.MONGODB;
			break;

		case PersistenceStrategy.POSTGRESQL:
			return PersistenceStrategy.POSTGRESQL;
			break;

		default:
			return PersistenceStrategy.MYSQL;
			break;
	}
};

export const getDomainOfPersistenceDataLayer = (): string => {
	const domainOfPersistenceDataLayer: Record<PersistenceStrategy, string> = {
		[PersistenceStrategy.MYSQL]: 'https://cloud.mysql.com',
		[PersistenceStrategy.MONGODB]: 'https://atlas.mongodb.com',
		[PersistenceStrategy.POSTGRESQL]: 'https://online.postgresql.com',
	};

	return domainOfPersistenceDataLayer[getPersistenceStrategy()];
};


/**
 * How to use this:
 *
 * process.env.PERSISTENCE_STRATEGY = 'mongodb'; // This should come from the .env file
 *
 * console.log(getDomainOfPersistenceDataLayer());
 */
```

----------------------------------------------------------
## Extendiendo la clase Error

Manera correcta de extender la clase Error en TypeScript: 
```typescript
/**
 * Declare an abstract class. All error classes should extend from this abstract factory.
 *
 * On this example, I added an optional property called "data". You can pass an Object to the constructor to add data to the error instance. Usefull to send data to a logger function or something like that. For example, you can use the property "message" to provide info for developers and use the property "data" to provide info to the API clients.
 */
export abstract class AbstractError extends Error {
	constructor(public readonly message: string, public readonly data?: Record<string, unknown>) {
		super(message);

		/* The next lines are necessary to use methods like 'instanceof' */
		this.name = this.constructor.name;
		Object.setPrototypeOf(this, new.target.prototype);
	}
}

/**
 * An implementation of AbstractError.
 */
export class FooError extends AbstractError {
	constructor(
		public readonly message: string = 'Default message error',
		public readonly data?: Record<string, unknown>,
	) {
		super(message);
	}
}

/**
 * Another implementation of AbstractError.
 */
export class BarError extends AbstractError {
	constructor(
		public readonly message: string = 'Default message error',
		public readonly data?: Record<string, unknown>,
	) {
		super(message);
	}
}

```

----------------------------------------------------------  
## Utilización de keyof

* Ejemplo interesante de _keyof_: 

```typescript
export type Person = {
	id: string;
  name: string;
  age: number;
  enabled: boolean;
};

const updateSomePerson = (
	timeSlotId: Person['id'], // allow the id of Person
	keyToUpdate: keyof Person, // allow any property of Person: 'id', 'name', 'age', 'enabled'
	newValue: Person[keyof Person] // allow string, number or boolean!
): void => {
	// ...
};

updateSomePerson('12a3b4c1', 'name', 'john doe');
```

----------------------------------------------------------  
## Utilización de Omit

* Evita hacer una propiedad opcional cuando la propiedad no es válida en un determinado caso. En su lugar, declara 2 tipos separados, y utiliza _Omit_ para evitar copiar/pegar.

Ejemplo: Muchos objetos carecen de id hasta que son guardados. Así que declara un tipo separado para la situación en que ese objeto esta "no guardado":

```typescript
type User = {
  id: number;
  name: string;
  email: string;
}

type UnsavedUser = Omit<User, 'id'>;
```

----------------------------------------------------------  

## Obtener los tipos de una librería de terceros no tipada


Are you ever building something in TypeScript and realize... AGH! This package is not exporting a type I need!

![Library not exporting types](./assets/library-not-exporting-types.jpeg)

Fortunately, TypeScript gives us a number of [utility types](https://www.typescriptlang.org/docs/handbook/utility-types.html) that can solve this common problem.

For instance, to grab the type returned from a function, we can use the `ReturnType` utility:

```typescript
import { getContent } from '@builder.io'
const content = await getContent()
// 😍
type Content = ReturnType<typeof getContent>
```

But we have one little problem. `getContent` is an `async` function that returns a promise, so currently our `Content` type is actually `Promise`, which is not what we want.

For that, we can use the `Awaited` type to unwrap the promise and get the type of what the promise resolves to:

```typescript
import { getContent } from '@builder.io'
const content = await getContent()
// ✅
type Content = Awaited<ReturnType<typeof getContent>>
```

Now we have exactly the type we needed, even though it is not explicitly exported. Well, that’s a relief.

But what if we need argument types for that function?

For instance, `getContent` takes an optional argument called `ContentKind` that is a union of strings. I really don’t want to have to type this out manually, so let’s use the `Parameters` utility type to extract its parameters:

```typescript
type Arguments = Parameters<typeof getContent>
// [ContentKind | undefined]
```
`Parameters` gives you a tuple of the argument types, and you can pull out a specific parameter type by index like so:

```typescript
type ContentKind = Parameters<typeof getContent>[0]
```
But we have one last issue. Because this is an optional argument, our `ContentKind` type right now is actually `ContentKind | undefined`, which is not what we want.

For this, we can use the `NonNullable` utility type, to exclude any `null` or `undefined` values from a union type.
```typescript
// ✅
type ContentKind = NonNullable<Parameters<typeof getContent>[0]>
// ContentKind
```
Now our `ContentKind` type perfectly matches the `ContentKind` in this package that was not being exported, and we can use it in our `processContent` function like so:
```typescript
import { getContent } from '@builder.io'

const content = await getContent()

type Content = Awaited<ReturnType<typeof getContent>>
type ContentKind = NonNullable<Parameters<typeof getContent>[0]>

// 🥳
function processContent(content: Content, kind: ContentKind) {
  // ...
}
```

----------------------------------------------------------  