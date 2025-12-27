# `src/infra/database/PgPromiseAdapter.ts`

## Papel na arquitetura

Adapter que conecta o sistema ao **PostgreSQL** utilizando `pg-promise`.

## Código original

```ts
import DatabaseConnection from "./DatabaseConnection";
import pgp from "pg-promise";

export default class PgPromiseAdapter implements DatabaseConnection {
	connection: any;

	constructor () {
		this.connection = pgp()("postgres://postgres:123456@localhost:5432/app");
	}

	query(statement: string, params: any): Promise<any> {
		return this.connection.query(statement, params);
	}

	close(): Promise<void> {
		return this.connection.$pool.end();
	}

}
```

## Anotações técnicas

- **Adapter Pattern**: traduz a API do `pg-promise` para a interface `DatabaseConnection`.
- **Conexão inicial**: string de conexão está fixa (poderia ser externalizada por env var).
- **Delegação**: `query` e `close` delegam ao driver real.

## Boas práticas

- **Isolamento de biblioteca externa**: facilita testes e troca de driver.
