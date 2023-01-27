# 📚 TypeScript: breves anotaciones sobre el lenguaje  

Listado personal de anotaciones, trucos, recordatorios, utilidades o ejemplos interesantes para TypeScript.  

![typescriptlang.png](./assets/typescriptlang.png) 

## Tabla de Contenido
- [Diccionarios](#diccionarios)
- [Mapeando opciones ](#mapeando-opciones)
- [Extendiendo la clase Error](#extendiendo-la-clase-error)



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

if (a.hasOwnProperty('age') && typeof a.age === 'number' && a.age > 25) { // TypeScript force you to check if property exists before using it
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