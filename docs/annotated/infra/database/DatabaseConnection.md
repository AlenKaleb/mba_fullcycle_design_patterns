# `src/infra/database/DatabaseConnection.ts`

## Papel na arquitetura

Interface para abstrair acesso a banco de dados, permitindo trocar drivers.

## Código original

```ts
export default interface DatabaseConnection {
	query (statement: string, params: any): Promise<any>;
	close (): Promise<void>;
}
```

## Anotações técnicas

- **Contrato mínimo**: define apenas operações necessárias (`query` e `close`).
- **Desacoplamento**: repositórios usam essa interface, não o driver concreto.

## Boas práticas

- **Inversão de dependência**: aplicação depende de abstrações, não de implementações.
