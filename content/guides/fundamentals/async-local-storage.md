De acordo com a [documentação oficial da Node.js](https://nodejs.org/docs/latest-v14.x/api/async_hooks.html): "`AsyncLocalStorage` é usado para criar um estado assíncrono dentro das funções de resposta e cadeias da promessa. **Este permite armazenar dados durante todo o tempo de vida duma requisição da Web ou qualquer outra duração assíncrona. É semelhante ao armazenamento local da linha em outras linguagens**".

Para simplificar ainda mais a explicação, `AsyncLocalStorage` permite-nos armazenar um estado quando executamos uma função assíncrona e depois o tornamos disponível a todos caminhos de código dentro desta função. Por exemplo:

:::note
O seguinte é um exemplo imaginário. No entanto, ainda podemos acompanhar o processo criando um projeto de Node.js vazio.
:::

Vamos criar uma instância do `AsyncLocalStorage` e exportá-lo a partir do seu módulo. Isto permitirá vários módulos acessarem a mesma instância do armazenamento:

```ts
// title: storage.ts
import { AsyncLocalStorage } from 'async_hooks'
export const storage = new AsyncLocalStorage()
```

Criamos o ficheiro principal. Este usará o método `storage.run` para executar uma função assíncrona com o estado inicial:

```ts
// title: main.ts
// highlight-start
import { storage } from './storage'
// highlight-end
import ModuleA from './ModuleA'

async function run(id) {
  // highlight-start
  const state = { id }

  return storage.run(state, async () => {
    await (new ModuleA()).run()
  })
  // highlight-end
}

run(1)
run(2)
run(3)
```

Finalmente, `ModuleA` pode acessar o estado usando o método `storage.getStore()`:

```ts
// title: ModuleA.ts
// highlight-start
import { storage } from './storage'
// highlight-end
import ModuleB from './ModuleB'

export default class ModuleA {
  public async run() {
    // highlight-start
    console.log(storage.getStore())
    await (new ModuleB()).run()
    // highlight-end
  }
}
```

Tal como `ModuleA`, o `ModuleB` também pode acessar o mesmo estado usando o método `storage.getStore`.

Em outras palavras, a cadeia inteira de operações tem acesso ao mesmo estado inicialmente definido dentro do ficheiro `main.js` durante a chamada do método `storage.run`.

## Qual é a Necessidade do Armazenamento Local Assíncrono?

<span id="#what-is-the-need-for-async-local-storage"></span>

Ao contrário de outras linguagens como a PHP, a Node.js não é uma linguagem processada em linha.

Na PHP, toda requisição de HTTP cria uma nova linha de processamento, cada linha de processamento tem sua memória. Isto permite-nos armazenar o estado numa memória global e acessá-lo em qualquer parte dentro da nossa base de código.

Na Node.js, não podemos guardar dados num objeto global e mantê-los isolados dentro das requisições da HTTP. Isto é impossível porque a Node.js executa numa única linha de processamento e partilha a memória entre todas as requisições de HTTP.

Isto é onde a Node.js ganha muito desempenho, uma vez que não tem inicializar a aplicação para cada requisição de HTTP.

No entanto, isto também significa que temos de passar o estado como argumentos de função ou argumentos de classe, uma vez que não podemos escrevê-lo ao objeto global. Algo como o seguinte:

```ts
http.createServer((req, res) => {
  const state = { req, res }
  await (new ModuleA()).run(state)
})

// Module A
class ModuleA {
  public async run(state) {
    await (new ModuleB()).run(state)
  }
}
```

> O armazenamento local assíncrono resolve este caso de uso, uma vez que permite o estado isolado entre várias operações assíncronas.

## How does AdonisJS uses ALS?

<span id="#how-does-adonisjs-uses-als"></span>

ALS stands for **AsyncLocalStorage**. AdonisJS uses the async local storage during the HTTP requests and set the [HTTP context](../http/context.md) as the state. The code flow looks similar to the following.

```ts
storage.run(ctx, () => {
  await runMiddleware()
  await runRouteHandler()
  ctx.finish()
})
```

The middleware and the route handler usually run other operations as well. For example, using a model to fetch the users.

```ts
export default class UsersController {
  public index() {
    await User.all()
  }
}
```

The `User` model instances now have access to the context since they are created within the code path of the `storage.run` method.

```ts
// highlight-start
import HttpContext from '@ioc:Adonis/Core/HttpContext'
// highlight-end

export default class User extends BaseModel {
  public get isFollowing() {
    // highlight-start
    const ctx = HttpContext.get()!
    return this.id === ctx.auth.user.id
    // highlight-end
  }
}
```

The model static properties (not methods) cannot access the HTTP context as they are evaluated when importing the model. So you must understand the code execution path and [use ALS carefully](#things-to-be-aware-of-when-using-als).

## Usage

<span id="#usage"></span>

To use ALS within your apps, you must enable it first inside the `config/app.ts` file. Feel free to create the property manually if it doesn't exist.

```ts
// title: config/app.ts
export const http: ServerConfig = {
  useAsyncLocalStorage: true,
}
```

Once enabled, you can access the current HTTP context anywhere inside your codebase using the `HttpContext` module.

:::note
Ensure the code path is called during the HTTP request for the `ctx` to be available. Otherwise, it will be `null`.
:::

```ts
import HttpContext from '@ioc:Adonis/Core/HttpContext'

class SomeService {
  public async someOperation() {
    const ctx = HttpContext.get()
  }
}
```

## How should it be used?

<span id="#how-should-it-be-used"></span>

At this point, you can consider Async Local Storage as a request-specific global state. [Global state or variables are generally considered bad](https://wiki.c2.com/?GlobalVariablesAreBad) as they make testing and debugging a lot harder.

Node.js Async Local Storage can get even trickier if you are not careful enough to access the local storage within the HTTP request.

We recommend you still write your code as you were writing earlier (passing `ctx` by reference), even if you have access to the Async Local Storage. Passing data by reference conveys a clear execution path and makes it easier to test your code in isolation.

### Then why have you introduced Async Local Storage?

<span id="#then-why-have-you-introduced-async-local-storage"></span>

Async Local Storage shines with APM tools, which collect performance metrics from your app to help you debug and pinpoint problems.

Before ALS, there was no simple way for APM tools to relate different resources with a given HTTP request. For example, It could show you how much time was taken to execute a given SQL query but could not tell you which HTTP request executed that query.

After ALS, now all this is possible without you have to touch a single line of code. **AdonisJS is going to use ALS to collect metrics using its application-level profiler**.

## Things to be aware of when using ALS

<span id="#things-to-be-aware-of-when-using-als"></span>

You are free to use ALS if you think it makes your code more straightforward and you prefer global access instead of passing everything by reference.

However, be aware of the following situations that can easily lead to memory leaks or unstable behavior of the program.

### Top-level access

<span id="#top-level-access"></span>

Never access the Async Local Storage at the top level of any module. For example:

#### ❌ Does not work

<span id="#-does-not-work"></span>

In Node.js, the modules are cached. So `HttpContext.get()` method will be executed only once during the first HTTP request and holds its `ctx` forever during the lifecycle of your process.

```ts
import HttpContext from '@ioc:Adonis/Core/HttpContext'
const ctx = HttpContext.get()

export default class UsersController {
  public async index() {
    ctx.request
  }
}
```

#### ✅ Works

<span id="#-works"></span>

Instead, you should move the `.get` call to within the `index` method.

```ts
export default class UsersController {
  public async index() {
    const ctx = HttpContext.get()
  }
}
```

### Inside static properties

<span id="#inside-static-properties"></span>

The static properties (not methods) of any class are evaluated as soon that module is imported, and hence you should not access the `ctx` within the static properties.

#### ❌ Does not work

<span id="#-does-not-work-1"></span>

In the following example, when you import the `User` model inside a controller, the `HttpContext.get()` code will be executed and cached forever. So either you will receive `null`, or you end up caching the tenant connection from the first request.

```ts
import HttpContext from '@ioc:Adonis/Core/HttpContext'

export default class User extends BaseModel {
  public static connection = HttpContext.get()!.tenant.connection
}
```

#### ✅ Works

<span id="#-works-1"></span>

Instead, you should move the `HttpContext.get` call to inside the `query` method.

```ts
import HttpContext from '@ioc:Adonis/Core/HttpContext'

export default class User extends BaseModel {
  public static query() {
    const ctx = HttpContext.get()!
    return super.query({ connection: tenant.connection })
  }
}
```

### Event handlers

<span id="#event-handlers"></span>

The handler of an event emitted during an HTTP request can get access to the request context using `HttpContext.get()` method. For example:

```ts
export default class UsersController {
  public async index() {
    const user = await User.create({})
    Event.emit('new:user', user)
  }
}
```

```ts
// title: Event handler
import HttpContext from '@ioc:Adonis/Core/HttpContext'

Event.on('new:user', () => {
  const ctx = HttpContext.get()
})
```

However, you should be aware of a couple of things when accessing the context from an event handler.

- The event must never try to send a response using `ctx.response.send()` because this is not what events are meant to do.
- Accessing `ctx` inside an event handler makes it rely on HTTP requests. In other words, the event is not generic anymore and should always be emitted during an HTTP request to make it work.
