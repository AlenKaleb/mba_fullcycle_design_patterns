# `src/infra/presenter/CsvPresenter.ts`

## Papel na arquitetura

Presenter que transforma a saída em **CSV** para integração com planilhas ou exportações.

## Código original

```ts
import { Output } from "../../application/usecase/GenerateInvoices";
import Presenter from "../../application/presenter/Presenter";
import moment from "moment";

export default class CsvPresenter implements Presenter {

	present(output: Output[]): any {
		const lines: any[] = [];
		for (const data of output) {
			const line: string[] = [];
			line.push(moment(data.date).format("YYYY-MM-DD"));
			line.push(`${data.amount}`);
			lines.push(line.join(";"));
		}
		return lines.join("\n");
	}

}
```

## Anotações técnicas

- **Formatação de datas**: `moment` padroniza a data em `YYYY-MM-DD`.
- **Separador `;`**: comum em CSVs locais (evita conflito com decimal usando vírgula).
- **Join por linhas**: cada `Output` vira uma linha; o resultado final é um texto.

## Boas práticas

- **Presenter especializado**: separa a responsabilidade de formatação da lógica de negócio.
