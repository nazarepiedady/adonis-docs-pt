---
summary: Saiba como definir e validar variáveis de ambiente na AdonisJS
---

No lugar de manter vários ficheiros de configuração, um para cada ambiente, a AdonisJS usa [variáveis de ambiente](https://en.wikipedia.org/wiki/Environment_variable) para valores que muitas vezes mudam entre o nosso ambiente local e de produção. Por exemplo: as credenciais de base de dados, um sinal booleano para alternar o armazenamento de consulta imediata de modelos de marcação, e assim por diante.

<span id="access-environment-variables"></span>

## Acessar as Variáveis de Ambiente

A Node.js permite-nos de maneira nativa acessar as variáveis de ambiente usando o objeto `process.env`. Por exemplo:

```ts
process.env.NODE_ENV
process.env.HOST
process.env.PORT
```

No entanto, recomendamos **usar o fornecedor de ambiente da AdonisJS**, visto que melhora mais a API para trabalhar com as variáveis de ambiente adicionando suporte para validações e fornecer informação de tipo estático:

```ts
import Env from '@ioc:Adonis/Core/Env'

Env.get('NODE_ENV')

// Com valores padrão
Env.get('HOST', '0.0.0.0')
Env.get('PORT', 3333)
```

<span id="why-validate-environment-variables"></span>

## Porquê Validar as Variáveis de Ambiente?

As variáveis de ambiente são injetadas de fora para dento da nossa aplicação, e temos pouco ou nenhum controlo sobre as mesmas dentro da nossa base de código.

Por exemplo, uma seção da nossa base de código depende da existência da variável de ambiente `SESSION_DRIVER`:

```ts
const driver = process.env.SESSION_DRIVER

// Código de reprodução
await Session.use(driver).read()
```

Não existe garantia de que no momento de executar o programa, a variável de ambiente `SESSION_DRIVER` exista e tenha o valor correto. Portanto devemos validá-la vs. receber um erro depois no ciclo de vida do programa reclamando sobre o valor **"undefined"**:

```ts
const driver = process.env.SESSION_DRIVER

if (!driver) {
  throw new Error('Missing env variable "SESSION_DRIVER"')
}

if (!['memory', 'file', 'redis'].includes(driver)) {
  throw new Error('Invalid value for env variable "SESSION_DRIVER"')  
}
```

Agora imagina escrever estas condições em toda parte dentro da nossa base de código? **Bem, não é uma excelente experiência de desenvolvimento**.

<span id="validating-environment-variables"></span>

## Validando Variáveis de Ambiente

A AdonisJS permite-nos **validar opcionalmente** as variáveis de ambiente desde muito cedo no ciclo de vida da inicialização da nossa aplicação e que recusa-se a iniciar se qualquer validação falhar.

Nós começamos por definir as regras de validação dentro do ficheiro `env.ts`:

```ts
import Env from '@ioc:Adonis/Core/Env'

export default Env.rules({
  HOST: Env.schema.string({ format: 'host' }),
  PORT: Env.schema.number(),
  APP_KEY: Env.schema.string(),
  APP_NAME: Env.schema.string(),
  CACHE_VIEWS: Env.schema.boolean(),
  SESSION_DRIVER: Env.schema.string(),
  NODE_ENV: Env.schema.enum(['development', 'production', 'test'] as const),
})
```

Além disto, a AdonisJS extrai a informação de tipo estático a partir das regras de validação e fornece sensor inteligente para as propriedades validadas.

![](https://res.cloudinary.com/adonis-js/image/upload/q_auto,f_auto/v1617158425/v5/adonis-env-intellisense.jpg)

<span id="schema-api"></span>

## API `Schema`

A seguir está a lista de métodos disponíveis para validar as variáveis de ambiente.

### `Env.schema.string`

Valida o valor para verificar se existe e se é uma sequência de caracteres válida. As sequências de caracteres vazias reprova as validações, devemos usar a variante opcional para permitir sequências de caracteres vazias:

```ts
{
  APP_KEY: Env.schema.string()
}

// Marcar como opcional
{
  APP_KEY: Env.schema.string.optional()
}
```

Nós também podemos forçar o valor à ter um dos formatos pré-definidos:

```ts
// Deve ser um hospedeiro válido (URL ou IP)
Env.schema.string({ format: 'host' })

// Deve ser uma URL válida
Env.schema.string({ format: 'url' })

// Deve ser um endereço de correio-eletrónico válido
Env.schema.string({ format: 'email' })
```

Quando validamos o formato de `url`, também podemos definir opções adicionais para forçar ou ignorar o `tdl` ou `protocol`:

```ts
Env.schema.string({ format: 'url', tld: false, protocol: false })
```

---

### `Env.schema.boolean`

Força o valor à ser uma representação de sequência de caracteres dum booleano. Os valores seguintes são considerados como booleanos válidos e serão convertidos para `true` ou `false`:

- `'1', 'true'` são moldados para `Boolean(true)`
- `'0', 'false'` são moldados para `Boolean(false)`

```ts
{
  CACHE_VIEWS: Env.schema.boolean()
}

// Marcar como opcional
{
  CACHE_VIEWS: Env.schema.boolean.optional()
}
```

---

### `Env.schema.number`

Força o valor à ser uma representação de sequência de caracteres válida dum número:

```ts
{
  PORT: Env.schema.number()
}

// Marcar como opcional
{
  PORT: Env.schema.number.optional()
}
```

---

### `Env.schema.enum`

Força o valor à ser um dos valores pré-definidos:

```ts
{
  NODE_ENV: Env
    .schema
    .enum(['development', 'production'] as const)
}

// Marcar como opcional
{
  NODE_ENV: Env
    .schema
    .enum
    .optional(['development', 'production'] as const)
}
```

---

<span id="custom-functions"></span>

### Funções Personalizadas

Para todo outro caso de uso de validação, podemos definir as nossas funções personalizadas:

```ts
{
  PORT: (key, value) => {
    if (!value) {
      throw new Error('Value for PORT is required')
    }
    
    if (isNaN(Number(value))) {
      throw new Error('Value for PORT must be a valid number')    
    }

    return Number(value)
  }
}
```

- Nós devemos certificar-nos de sempre retornar o valor depois de validá-lo.
- O valor de retorno pode ser diferente do valor da entrada inicial.
- Nós inferimos o tipo estático a partir do valor de retorno. Neste caso, `Env.get('PORT')` é um número.

<span id="defining-variables-in-the-development"></span>

## Definindo Variáveis no Desenvolvimento

Durante o desenvolvimento, podemos definir variáveis de ambiente dentro do ficheiro `.env` armazenado na raiz do nosso projeto, e a AdonisJS irá processá-lo automaticamente:

```dotenv
// title: .env
PORT=3333
HOST=0.0.0.0
NODE_ENV=development
APP_KEY=sH2k88gojcp3PdAJiGDxof54kjtTXa3g
SESSION_DRIVER=cookie
CACHE_VIEWS=false
```

<span id="variable-substitution"></span>

### Substituição de Variável

Junto com o suporte padrão para analise do ficheiro `.env`, a AdonisJS também permite a substituição de variável:

```sh
HOST=localhost
PORT=3333
// highlight-start
URL=$HOST:$PORT
// highlight-end
```

Todos os `letter`, `numbers`, e sublinhados (`_`) depois do sinal (`$`) do dólar são analisados como variáveis. Se a nossa variável contiver qualquer outro carácter, então devemos envolvê-la dentro das chavetas `{}`:

```sh
REDIS-USER=foo
// highlight-start
REDIS-URL=localhost@${REDIS-USER}
// highlight-end
```

<span id="escape-the--sign"></span>

### Escapar o Sinal `$`

Se o valor duma variável contiver um sinal `$`, devemos escapa-lo para evitar a substituição de variável: 

```sh
PASSWORD=pa\$\$word
```

<span id="do-not-commit-the-env-file"></span>

### Não Consolidar o Ficheiro `.env`

Os ficheiros `.env` não portáteis. Querendo dizer que, as credenciais da base de dados no nosso ambiente local e de produção sempre serão diferentes, por isto não precisamos de empurrar o `.env` para o controlo de versão.

Nós devemos considerar o ficheiro `.env` pessoal ao nosso ambiente local e criar um ficheiro `.env` separado no servidor de produção ou de encenação (e mantê-lo seguro).

O ficheiro `.env` pode estar em qualquer localização no nosso servidor. Por exemplo, podemos armazená-lo dentro de `/etc/myapp/.env` e depois informar a AdonisJS sobre ele como se segue:

```sh
ENV_PATH=/etc/myapp/.env node server.js
```

<span id="defining-variables-during-tests"></span>

## Definindo Variáveis Durante os Testes

A AdonisJS procurará pelo ficheiro `.env.test` quando a aplicação for iniciada com a variável de ambiente `NODE_ENV=test`.

As variáveis definidas dentro do ficheiro `.env.test` são automaticamente combinadas com o ficheiro `.env`. Isto permite-nos usar uma base de dados diferente ou um condutor de sessão diferente quando escrevemos testes.

<span id="defining-variables-in-production"></span>

## Definindo Variáveis em Produção

A maioria dos provedores de hospedagem modernos têm suporte de primeira classe para definição de variáveis de ambiente dentro da sua consola da Web. Nós devemos certificar-nos de ler a documentação para o nosso provedor de hospedagem e definir as variáveis de ambiente antes de implementar a nossa aplicação de AdonisJS em produção.
