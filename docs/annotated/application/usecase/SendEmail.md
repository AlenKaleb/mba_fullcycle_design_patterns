# `src/application/usecase/SendEmail.ts`

## Papel na arquitetura

Caso de uso simples que representa o **envio de e-mails** após geração de faturas. Aqui está apenas como *stub*.

## Código original

```ts
export default class SendEmail {

	constructor () {
	}

	async execute (input: any): Promise<void> {
		console.log(input);
	}
}
```

## Anotações técnicas

- **Stub/placeholder**: não envia e-mail real, apenas loga o input.
- **Separação por caso de uso**: demonstra como side effects podem ser encapsulados fora do domínio.

## Boas práticas

- **Facilidade de evolução**: implementação real pode ser adicionada sem alterar `GenerateInvoices`.
