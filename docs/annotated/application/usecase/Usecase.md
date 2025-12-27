# `src/application/usecase/Usecase.ts`

## Papel na arquitetura

Interface base para casos de uso na camada de aplicação, padronizando a execução.

## Código original

```ts
export default interface Usecase {
	execute (input: any): Promise<any>;
}
```

## Anotações técnicas

- **Contrato uniforme**: todos os casos de uso expõem `execute` com retorno assíncrono.
- **Flexibilidade**: `any` permite variar tipos de input/output conforme o caso de uso.

## Boas práticas

- **Consistência**: facilita composição (ex.: decorators) e integração com controllers.
