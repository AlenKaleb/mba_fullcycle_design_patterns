# `src/domain/InvoiceGenerationStrategy.ts`

## Papel na arquitetura

Contrato do **Strategy Pattern** para geração de faturas. Define a interface que todas as estratégias devem implementar.

## Código original

```ts
import Contract from "./Contract";
import Invoice from "./Invoice";

export default interface InvoiceGenerationStrategy {
	generate (contract: Contract, month: number, year: number): Invoice[];
}
```

## Anotações técnicas

- **Interface explícita**: fixa o formato da operação `generate`.
- **Baixo acoplamento**: o domínio pode variar a implementação sem alterar quem chama.

## Boas práticas

- **Substituibilidade (LSP)**: qualquer estratégia pode ser usada onde o tipo é esperado.
