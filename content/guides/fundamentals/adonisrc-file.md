---
previewCode: >
  // visualizar o conteúdo e padrões do ficheiro.
  node ace dump:rcfile
summary: O ficheiro adonisrc.json configura o espaço de trabalho e algumas definições de execução duma aplicação de AdonisJS. Ele também permite-te sobrepor as convenções padrão em torno da estrutura de ficheiro.
---

O ficheiro `.adonisrc.json` é armazenado dentro da raiz do teu projeto. Ele configura o espaço de trabalho e algumas definições de execução da tua aplicação de AdonisJS.

O ficheiro contém apenas a mínima configuração exigida para executares a tua aplicação. No entanto, podes visualizar o conteúdo completo do ficheiro executando o seguinte comando:

```sh
node ace dump:rcfile
```

```json
// title: Output
{
  "typescript": true,
  "directories": {
    "config": "config",
    "public": "public",
    "contracts": "contracts",
    "providers": "providers",
    "database": "database",
    "migrations": "database/migrations",
    "seeds": "database/seeders",
    "resources": "resources",
    "views": "resources/views",
    "start": "start",
    "tmp": "tmp",
    "tests": "tests"
  },
  "exceptionHandlerNamespace": "App/Exceptions/Handler",
  "preloads": [
    {
      "file": "./start/routes",
      "optional": false,
      "environment": [
        "web",
        "console",
        "test"
      ]
    },
    {
      "file": "./start/kernel",
      "optional": false,
      "environment": [
        "web",
        "console",
        "test"
      ]
    },
    {
      "file": "./start/views",
      "optional": false,
      "environment": [
        "web",
        "console",
        "test"
      ]
    },
    {
      "file": "./start/events",
      "optional": false,
      "environment": [
        "web"
      ]
    }
  ],
  "namespaces": {
    "models": "App/Models",
    "middleware": "App/Middleware",
    "exceptions": "App/Exceptions",
    "validators": "App/Validators",
    "httpControllers": "App/Controllers/Http",
    "eventListeners": "App/Listeners",
    "redisListeners": "App/Listeners"
  },
  "aliases": {
    "App": "app",
    "Config": "config",
    "Database": "database",
    "Contracts": "contracts"
  },
  "metaFiles": [
    {
      "pattern": "public/**",
      "reloadServer": false
    },
    {
      "pattern": "resources/views/**/*.edge",
      "reloadServer": false
    }
  ],
  "commands": [
    "./commands",
    "@adonisjs/core/build/commands",
    "@adonisjs/repl/build/commands"
  ],
  "commandsAliases": {
  },
  "tests": {
    "suites": [
      {
        "name": "functional",
        "files": [
          "tests/functional/**/*.spec.ts"
        ],
        "timeout": 30000
      }
    ]
  },
  "providers": [
    "./providers/AppProvider",
    "@adonisjs/core",
    "@adonisjs/session",
    "@adonisjs/view"
  ],
  "aceProviders": [
    "@adonisjs/repl"
  ],
  "testProviders": [
    "@japa/preset-adonis/TestsProvider"
  ]
}
```

### `typescript`

A propriedade `typescript` informa para abstração e os comandos de `ace` que a tua aplicação está usando TypeScript. Atualmente, este valor é sempre definido para `true`. No entanto, permitiremos depois as aplicações serem escritas em JavaScript também.

---

### `directories`

Um objeto de diretórios conhecidos e os seus caminhos pré-configurados. Tu podes mudar o caminho para corresponder aos teus requisitos.

Além disto, todos os comandos `make` de `ace` fazem referência ao ficheiro `.adonisrc.json` antes de criarem o ficheiro:

```json
{
  "directories": {
    "config": "config",
    "public": "public",
    "contracts": "contracts",
    "providers": "providers",
    "database": "database",
    "migrations": "database/migrations",
    "seeds": "database/seeders",
    "resources": "resources",
    "views": "resources/views",
    "start": "start",
    "tmp": "tmp",
    "tests": "tests"
  }
}
```

---

### `exceptionHandlerNamespace`

O espaço de nome para a classe que manipula as exceções ocorridas durante uma requisição de HTTP:

```json
{
  "exceptionHandlerNamespace": "App/Exceptions/Handler"
}
```

---

### `preloads`

Um arranjo de ficheiros para carregar no momento de inicializar a aplicação. Os ficheiros são carregados imediatamente depois de inicializar os provedores de serviços.

Tu podes definir o ambiente no qual carregar o ficheiro. As opções válidas são:

- `web`: refere-se ao processo iniciado para o servidor de HTTP.
- `console`: refere-se aos comandos de `ace` exceto para o comando `repl`.
- `repl`: refere-se ao processo iniciado usando o comando `node ace repl`.
- `test`: é reservado para o futuro quando a AdonisJS ter o executor de teste embutido.

Além disto, podes marcar o ficheiro como opcional, e o ignoraremos se o ficheiro estiver em falta no disco.

:::note

Tu podes criar e registar um ficheiro pré-carregado executando o comando `node ace make:prldfile`.

:::

```json
{
  "preloads": [
    {
      "file": "./start/routes",
      "optional": false,
      "environment": [
        "web",
        "console",
        "test"
      ]
    },
  ]
}
```

---

### `namespaces`

Um objeto de espaços de nome para entidades conhecidas.

Por exemplo, podes mudar o espaço de nome do controlador de `App/Controllers/Http` para `App/Controllers` e manter os controladores dentro do diretório `./app/Controllers`:

```json
{
  "namespaces": {
    "controllers": "App/Controllers"
  }
}
```

---

### `aliases`

A propriedade `aliases` permite-te definir os pseudónimos de importação para diretórios específicos. Depois de definir o pseudónimo, serás capaz de importar os ficheiros a partir da raiz do diretório de pseudónimos.

No seguinte exemplo, o `App` é um pseudónimo para o diretório `./app`, e o resto é o caminho de ficheiro a partir do dado diretório:

```ts
import 'App/Models/User'
```

Os pseudónimos da AdonisJS são apenas para o momento da execução. Tu também terás de registar o mesmo pseudónimo dentro do ficheiro `tsconfig.json` para o compilador de TypeScript funcionar:

```json
{
  "compilerOptions": {
    "paths": {
      "App/*": [
        "./app/*"
      ],
    }
  }
}
```

---

### `metaFiles`

O arranjo `metaFiles` aceita os ficheiros que quiseres que a AdonisJS copie para a pasta `build` quando criar a construção de produção.

- Tu podes definir os caminhos de ficheiro como um padrão glob, e copiaremos todos os ficheiros correspondentes para este padrão.
- Tu podes também instruir o servidor de desenvolvimento a recarregar quaisquer ficheiros dentro das mudanças do padrão correspondente.

```json
{
  "metaFiles": [
    {
      "pattern": "public/**",
      "reloadServer": false
    },
    {
      "pattern": "resources/views/**/*.edge",
      "reloadServer": false
    }
  ]
}
```

---

### `commands`

Um arranjo de caminhos para procurar ou indexar comandos de `ace`. Tu podes definir um caminho relativo como `./command` ou caminho para um pacote instalado:

```json
{
  "commands": [
    "./commands",
    "@adonisjs/core/build/commands"
  ]
}
```

---

### `commandsAliases`

Um par de chave-valor de pseudónimos de comando. Isto é normalmente para ajudar-te a criar pseudónimos memorizáveis para os comandos que são mais difíceis de digitar ou lembrar:

```json
{
  "commandsAliases": {
    "migrate": "migration:run"
  }
}
```

Tu também podes definir vários pseudónimos adicionado várias entradas:

```json
{
  "commandsAliases": {
    "migrate": "migration:run",
    "up": "migration:run"
  }
}
```

---

### `tests`

O objeto `test` segura a coleção de grupos de teste usada pela tua aplicação. Tu podes adicionar ou remover os grupos de acordo com os requisitos da tua aplicação:

```json
{
 "tests": {
    "suites": [
      {
        "name": "functional",
        "files": [
          "tests/functional/**/*.spec.ts"
        ],
        "timeout": 30000
      }
    ]
  }
}
```

---

### `providers`

Um arranjo de provedores de serviço à carregar durante o ciclo de inicialização da aplicação. Os provedores mencionados dentro deste arranjo são carregados em todos os ambientes:

```json
{
  "providers": [
    "./providers/AppProvider",
    "@adonisjs/core"
  ],
}
```

---

### `aceProviders`

Um arranjo de provedores exigido pelos comandos de `ace`:

```json
{
  "aceProviders": [
    "@adonisjs/repl"
  ]
}
```

---

### `testProviders`

Um arranjo de provedores carregados apenas durante os testes:

```json
{
  "testProviders": [
    "@japa/preset-adonis/TestsProvider"
  ]
}
```
