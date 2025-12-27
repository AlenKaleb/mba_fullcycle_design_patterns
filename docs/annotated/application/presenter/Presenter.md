# `src/application/presenter/Presenter.ts`

## Papel na arquitetura

Interface para **formatação da saída** dos casos de uso.

## Código original

```ts
import { Output } from "../usecase/GenerateInvoices";

export default interface Presenter {
	present (output: Output[]): any;
}
```

## Anotações técnicas

- **Separação de apresentação**: a lógica de formato (JSON, CSV) fica fora do caso de uso.
- **Tipagem**: baseia-se em `Output` do caso de uso `GenerateInvoices`.

## Boas práticas

- **Open/Closed**: novos formatos podem ser adicionados via novas implementações.
