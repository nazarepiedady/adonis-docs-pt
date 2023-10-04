---
summary: O módulo de aplicação da AdonisJS é responsável pela inicialização e gestão do seu ciclo de vida.
---

O módulo de aplicação da AdonisJS é responsável pela inicialização da aplicação em diferentes ambientes conhecidos.

Quando inicias o servidor HTTP a partir do ficheiro `server.ts` ou executas o comando `node ace serve`, a aplicação é inicializada para o ambiente **web**.

Ao passo que a execução do comando `node ace repl` inicia a aplicação no ambiente **repl**. Todos outros comandos iniciam a aplicação no ambiente **console**.

O ambiente da aplicação desempenha uma papel essencial em decidir quais ações realizar. Por exemplo, o ambiente **web** não regista ou inicializa os provedores de Ace.

Nós podemos acessar o ambiente atual da aplicação usando a propriedade `environment`. A seguir está a lista dos ambientes de aplicação conhecidos:

- `web`: o ambiente refere-se ao processo iniciado para o servidor de HTTP.
- `console`: o ambiente refere-se aos comandos do Ace exceto para o comando REPL.
- `repl`: o ambiente refere-se aos processos iniciado com o uso do comando `node ace repl`.
- `test`: o ambiente refere-se ao processo iniciado com o uso do comando `node ace test`.

```ts
import Application from '@ioc:Adonis/Core/Application'
console.log(Application.environment)
```

<span id="boot-lifecycle"></span>

## Ciclo de Vida da Inicialização

A seguir está o ciclo de inicialização da aplicação.

::img[]{src="https://res.cloudinary.com/adonis-js/image/upload/q_auto,f_auto/v1617132548/v5/application-boot-lifecycle.png" width="300px"}

<!--

graph TD
A[state:unknown] - ->|Inicia o processo de inicialização| B(state:initiated)
B - ->|Carrega e valida as variáveis de ambiente<br />Carrega a configuração<br />Configura o registador e o perfilador | C(state:setup)
C - ->|Regista os provedores| D(state:registered)
D - ->|Inicializa os provedores <br />Importa ficheiros pré-carregados| E(state:booted)
E - ->|Executa o método pronto de provedores| F(state:ready)
F - -> G[[Run main code]]
G - -> H[/Desliga o método invocado/]
H - ->|Executa o método de desligamento de provedores| I(state:shutdown)

-->

Nós podemos acessar o contentor de vinculações IoC, uma vez que o estado da aplicação for definido para `booted` ou `ready`. Uma tentativa para acessar o contentor de vinculações antes do estado inicializado resulta em uma exceção.

Por exemplo, se tiveres um provedor de serviço que queira resolver as vinculações do contentor, deverias escrever a declaração importação dentro dos métodos `boot` ou `ready`.

:::caption{for="error"}
A importação de alto nível não funcionará.
:::

```ts
import { ApplicationContract } from '@ioc:Adonis/Core/Application'
import Route from '@ioc:Adonis/Core/Route'

export default class AppProvider {
  constructor(protected app: ApplicationContract) {}

  public async boot() {
    Route.get('/', async () => {})
  }
}
```

:::caption{for="success"}
Mova a importação para dentro do método de inicialização.
:::

```ts
import { ApplicationContract } from '@ioc:Adonis/Core/Application'

export default class AppProvider {
  constructor(protected app: ApplicationContract) {}

  public async boot() {
    const { default: Route } = await import('@ioc:Adonis/Core/Route')
    Route.get('/', async () => {})
  }
}
```

<span id="version"></span>

## Versão

Nós podemos acessar a versão da aplicação e da abstração usando as propriedades `version` e `adonisVersion`.

A propriedade `version` refere-se à versão dentro do ficheiro `package.json` da sua aplicação. A propriedade `adonisVersion` refere-se à versão instalada do pacote `@adonisjs/core`:

```ts
import Application from '@ioc:Adonis/Core/Application'

console.log(Application.version!.toString())
console.log(Application.adonisVersion!.toString())
```

Ambas as propriedades de versão são representadas como um objeto com as sub-propriedades `major`, `minor`, `patch`.

```ts
console.log(Application.version!.major)
console.log(Application.version!.minor)
console.log(Application.version!.patch)
```

<span id="node-environment"></span>

## Ambiente da Node

Nós podemos acessar o ambiente `node` usando a propriedade `nodeEnvironment`. O valor é uma referência à variável `NODE_ENV`. No entanto, o valor será normalizado mais adiante para ser consistente:

```ts
import Application from '@ioc:Adonis/Core/Application'

console.log(Application.nodeEnvironment)
```

| NODE_ENV | Normalizado para | 
|------------|----------------|
| dev | desenvolvimento |
| develop | desenvolvimento |
| stage | encenação |
| prod | produção |
| testing | teste |

Além disto, podes fazer uso das seguintes propriedades como atalho para saber qual é o ambiente atual.

### `inProduction`

```ts
Application.inProduction

// O mesmo que
Application.nodeEnvironment === 'production'
```

---

### `inDev`

```ts
Application.inDev

// O mesmo que
Application.nodeEnvironment === 'development'
```

---

### `inTest`

```ts
Application.inTest

// O mesmo que
Application.nodeEnvironment === 'test'
```

<span id="make-paths-to-project-directories"></span>

## Criar Caminhos para os Diretórios do Projeto

Nós podemos fazer uso do módulo de aplicação para criar caminhos absolutos para diretórios conhecidos do projeto.

### `configPath`

Cria um caminho absoluto para um ficheiro dentro do diretório `config`:

```ts
Application.configPath('shield.ts')
```

---

### `publicPath`

Cria um caminho absoluto para um ficheiro dentro do diretório `public`:

```ts
Application.publicPath('style.css')
```

---

### `databasePath`

Cria um caminho absoluto para um ficheiro dentro do diretório `database`:

```ts
Application.databasePath('seeders/Database.ts')
```

---

### `migrationsPath`

Cria um caminho absoluto para um ficheiro dentro do diretório `migrations`:

```ts
Application.migrationsPath('users.ts')
```

---

### `seedsPath`

Cria um caminho absoluto para um ficheiro dentro do diretório `seeds`:

```ts
Application.seedsPath('Database.ts')
```

---

### `resourcesPath`

Cria um caminho absoluto para um ficheiro dentro do diretório `resources`:

```ts
Application.resourcesPath('scripts/app.js')
```

---

### `viewsPath`

Cria um caminho absoluto para um ficheiro dentro do diretório `views`:

```ts
Application.viewsPath('welcome.edge')
```

---

### `startPath`

Cria um caminho absoluto para um ficheiro dentro do diretório `start`:

```ts
Application.startPath('routes.ts')
```

---

### `tmpPath`

Cria um caminho absoluto para um ficheiro dentro do diretório `tmp`:

```ts
Application.tmpPath('uploads/avatar.png')
```

---

### `makePath`

Cria um caminho absoluto a partir da raiz da aplicação:

```ts
Application.makePath('app/Middleware/Auth.ts')
```

## Outras Propriedades

A seguir está a lista de propriedades no módulo de aplicação.

### `appName`

Nome da aplicação. Ela refere-se a propriedade `name` dentro do ficheiro `package.json` da sua aplicação:

```ts
Application.appName
```

---

### `appRoot`

Caminho absoluto para o diretório raiz da aplicação:

```ts
Application.appRoot
```

---

### `rcFile`

Refere-se ao [ficheiro `adonisrc`](./adonisrc-file) analisado:

```ts
Application.rcFile.providers
Application.rcFile.raw
```

---

### `container`

Refere-se ao instância do contentor IoC:

```ts
Application.container
```

---

### `helpers`

Refere-se ao módulo de auxiliares:

```ts
Application.helpers.string.snakeCase('helloWorld')
```

Nós podemos também acessar os módulos auxiliares diretamente:

```ts
import { string } from '@ioc:Adonis/Core/Helpers'

string.snakeCase('helloWorld')
```

---

### `logger`

Refere-se ao registador de aplicação:

```ts
Application.logger.info('hello world')
```

Nós podemos também acessar o módulo registador diretamente:

```ts
import Logger from '@ioc:Adonis/Core/Logger'

Logger.info('hello world')
```

---

### `config`

Refere-se ao módulo de configuração:

```ts
Application.config.get('app.secret')
```

Nós podemos acessar o módulo de configuração diretamente:

```ts
import Config from '@ioc:Adonis/Core/Config'

Config.get('app.secret')
```

---

### `env`

Refere-se ao módulo de ambiente:

```ts
Application.env.get('APP_KEY')
```

Nós podemos também acessar o módulo de ambiente diretamente:

```ts
import Env from '@ioc:Adonis/Core/Env'

Env.get('APP_KEY')
```

---

### `isReady`

Informa se a aplicação está no estado preparada. Ela é usada internamente para parar de aceitar novas requisições de HTTP quando a `isReady` é falsa:

```ts
Application.isReady
```

---

### `isShuttingDown`

Informa se a aplicação está no processo de desligamento:

```ts
Application.isShuttingDown
```
