---
summary: O módulo de configuração para gerir e fornecer a configuração da aplicação aos pacotes instalados.
---

A configuração de tempo de execução na tua aplicação de AdonisJS é guardada dentro do diretório `config`. O núcleo da abstração e muitos dos pacotes instalados dependem destes ficheiros de configuração. Então certifica-te de passar pelos ficheiros de configuração e fazer pequenas melhorias em quaisquer definições (se necessário).

Nós também recomendamos guardar todos as configurações personalizados exigidas pela tua aplicação dentro deste diretório contra guardá-los em vários lugares.

## Importar os Ficheiros Configuração

Tu podes importar os ficheiros de configuração dentro da base de código da tua aplicação usando a declaração `import`. Por exemplo:

```ts
import { appKey } from 'Config/app'
```

## Usando o Provedor de Configuração


No lugar de importar diretamente os ficheiros de configuração, também podes fazer uso do provedor `Config` como se segue:

```ts
import Config from '@ioc:Adonis/Core/Config'

Config.get('app.appKey')
```

O método `Config.get` aceita um caminho separado por ponto para a chave da configuração. No exemplo acima, lemos a propriedade `appKey` a partir do ficheiro `config/app.ts`.

Além disto, podes definir um valor de retorno. O valor de retorno é retornado quando o valor da configuração verdadeira estiver em falta:

```ts
Config.get('database.connections.mysql.host', '127.0.0.1')
```

Não existe nenhum beneficio direto em usar o provedor `Config` sobre a importação manual dos ficheiros de configuração. No entanto, o provedor `Config` é a única escolha nos seguintes cenários.

- **Pacotes externos**: Pacotes externos nunca devem depender do caminho do ficheiro para ler e importar a configuração. Ao invés disso, deve fazer uso do provedor `Config`. O uso do provedor `Config` cria um acoplamento solto entre a aplicação e o pacote.
- **Modelos de marcação de Edge**: Os ficheiros de modelos de marcação podem usar o método de [configuração](../../reference/views/globals/all-helpers#config) global para referirem-se aos valores de configuração.

## Mudando a Localização da Configuração

Tu podes atualizar a localização para o diretório de configuração modificando o ficheiro `.adonisrc.json`:

```json
"directories": {
  "config": "./configurations"
}
```

O provedor de configuração lerá automaticamente o ficheiro a partir do diretório configurado recentemente, e todos os pacotes subjacentes que dependem dos ficheiros de configuração funcionarão perfeitamente.

## Advertências

Todos os ficheiros de configuração dentro do diretório `config` são importados automaticamente pela abstração durante a fase de inicialização. Como resultado disto, os teus ficheiros de configuração não deveriam depender das ligações do contentor.

Por exemplo, o seguinte código quebrará visto que tenta importar o provedor `Route` mesmo antes de ser registado ao contentor.

```ts
// ❌ Não funciona
import Route from '@ioc:Adonis/Core/Route'

const someConfig = {
  assetsUrl: Route.makeUrl('/assets')
}
```

Tu podes considerar esta limitação algo mau. No entanto, tem um impacto positivo sobre o desenho da aplicação.

Fundamentalmente, o teu código de tempo de execução deveria depender da configuração e NÃO o contrário. Por exemplo:

:::caption{for="error"}
Não derive a configuração a partir do código de tempo de execução (Modelo neste caso).
:::
```ts
import User from 'App/Models/User'

const someConfig = {
  databaseTable: User.table
}
```

:::caption{for="success"}
No lugar disto que está acima, faça o teu modelo ler a tabela a partir do ficheiro de configuração.
:::

```ts
const someConfig = {
  databaseTable: 'users'
}
```

```ts
import someConfig from 'Config/file/path'

class User extends Model {
  public static table = someConfig.databaseTable
}
```

## Referência da Configuração

Conforme instalares e configurares os pacotes de AdonisJS, eles podem criar novos ficheiros de configuração. A seguir está uma lista de ficheiros de configuração (com seus moldes padrão) usado pelas diferentes partes da abstração.

| Ficheiro de Configuração | Talão | Usado pelo |
|------------|------|----------|
| `app.ts` | https://git.io/JfefZ | Usado pelo núcleo da abstração, incluindo o servidor HTTP, registador de relatório, validador, e o gestor de recursos. |
| `bodyparser.ts` | https://git.io/Jfefn | Usado pelo intermediário `bodyparser` |
| `cors.ts` | https://git.io/JfefC | Usado pelo gatilho de servidor CORS |
| `hash.ts` | https://git.io/JfefW | Usado pelo pacote de `hash` |
| `session.ts` | https://git.io/JeYHp | Usado pelo pacote de sessão |
| `shield.ts` | https://git.io/Jvwvt | Usado pelo pacote de proteção
| `static.ts` | https://git.io/Jfefl | Usado pelo servidor de ficheiro estático |
| `auth.ts` | https://git.io/JY0mp | Usado pelo pacote de autenticação |
| `database.ts` | https://git.io/JesV9 | Usado pela ORM Lucid |
| `mail.ts` | https://git.io/JvgAf | Usado pelo pacote de correio da AdonisJS |
| `redis.ts` | https://git.io/JemcF | Usado pelo pacote de Redis |
| `drive.ts` | https://git.io/JBt3o | Usado pelo provedor de Drive |
| `ally.ts` | https://git.io/JOdi5 | Usado pelo pacote de autenticação Social (Ally) |
