---
summary: Saiba como implementar as tuas aplicações de AdonisJS num servidor de produção.
---

A implementação duma aplicação de AdonisJS em produção não é diferente da implementação de qualquer outra aplicação de Node.js em produção. Primeiro, precisamos dum servidor ou duma plataforma que pode instalar e executar o lançamento mais recente da `Node.js v14`.

:::note
Para uma experiência de implementação sem tensões, podemos tentar a Cleavr. É um serviço de provisão de servidor e tem suporte de primeira classe para [implementação de aplicações de AdonisJS em produção](https://cleavr.io/adonis).

**Desmentido: Cleavr também uma patrocinadora da AdonisJS.**
:::

<span id="compiling-typescript-to-javascript"></span>

## Compilando a TypeScript para JavaScript

As aplicações da AdonisJS são escritas em TypeScript e devem ser compiladas para JavaScript durante a implementação em produção. Nós podemos compilar a nossa aplicação diretamente no servidor de produção ou executar uma fase de construção numa conduta de CI/CD.

Nós podemos construir o nosso [código para produção](./typescript-build-process.md#standalone-production-builds) executando o seguinte comando de `ace`. O saída de JavaScript compilada é escrita dentro do diretório `build`:

```sh
node ace build --production
```

Se executamos a fase de construção dentro duma conduta CI/CD, então podemos apenas mover a pasta `build` para o nosso servidor de produção e instalar as dependências de produção diretamente no servidor.

<span id="starting-the-production-server"></span>

## Iniciando o Servidor de Produção

Nós podemos iniciar o servidor de produção executando o ficheiro `server.js`.

Se executamos a fase de construção no nosso servidor de produção, temos de certificar-nos de primeiro entrar dentro do diretório `build` e então iniciar o servidor.

```sh
cd build
npm ci --production

# Iniciar o servidor
node server.js
```

Se a fase de construção foi executada numa conduta de CI/CD e **copiamos apenas a pasta `build` para no nosso servidor de produção**, o diretório `build` torna-se a raiz da nossa aplicação:

```sh
npm ci --production

# Iniciar o servidor
node server.js
```

<span id="using-a-process-manager"></span>

### Usando um Gestor de Processo

É recomendado usar um gestor de processo quando gerimos uma aplicação de Node.js num servidor básico.

Um gestor de processo assegura-se de reiniciar a aplicação se avariar durante o tempo de execução. Além disso, alguns gestores de processo como o [PM2](https://pm2.keymetrics.io/docs/usage/quick-start/) também podem realizar reinicializações graciosas quando re-implementamos a aplicação em produção.

A seguir está um [ficheiro de ecossistema](https://pm2.keymetrics.io/docs/usage/application-declaration/) de exemplo para o PM2:

```ts
module.exports = {
  apps: [
    {
      name: 'web-app',
      script: './build/server.js',
      instances: 'max',
      exec_mode: 'cluster',
      autorestart: true,
    },
  ],
}
```

<span id="nginx-reverse-proxy"></span>

## Delegação Reversa de NGINX

Quando executamos a aplicação de AdonisJS num servidor básico, devemos colocá-la por trás da NGINX (ou servidor da Web semelhante) por [muitas diferentes razões](https://medium.com/intrinsic/why-should-i-use-a-reverse-proxy-if-node-js-is-production-ready-5a079408b2ca), com a terminação de SSL sendo uma das mais importantes.

:::note
Certifica-te de ler o [guia de delegações de confiança](../http/request.md#trusted-proxy) para assegurares-te de podes acessar o endereço de IP correto do visitante quando executares a aplicação de AdonisJS por trás dum servidor de delegação.
:::

A seguir está um exemplo de configuração de NGINX para delegar requisição à nossa aplicação de AdonisJS. **Temos de certificar-nos de substituir os valores dentro dos parênteses angulares `<>`**:

```nginx
server {
  listen 80;

  server_name <APP_DOMAIN.COM>;

  location / {
    proxy_pass http://localhost:<ADONIS_PORT>;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```

<span id="migrating-database"></span>

## Migrando a Base de Dados

Usando o comando `node ace migration:run --force`, podemos migrar a nossa base de dados de produção. A opção `--force` é obrigatória quando executamos migrações no ambiente de produção.

<span id="when-migrate"></span>

### Quando Migrar

Além disto, seria melhor se sempre que executarmos as migrações antes de reiniciar o servidor. Então, se a migração falhar, não reiniciar o servidor.

Usando um serviço administrado como Cleavr ou Heroku, podem lidar com este caso de uso automaticamente. De outro modo, teremos de executar o programa de migração dentro duma conduta de CI/CD ou executá-lo manualmente através de SSH.

<span id="do-not-rollback-in-production"></span>

### Não Recuar em Produção

O método `down` nos nossos ficheiros de migração normalmente contém ações destrutivas como **eliminar a tabela**, ou **remover uma coluna**, e por aí fora. É recomendado desligar retrocessos em produção dentro do ficheiro `config/database.ts`.

Desligar os retrocessos em produção garantirá que o comando `node ace migration:rollback` resultará num erro:

```ts
{
  pg: {
    client: 'pg',
    migrations: {
      // highlight-start
      disableRollbacksInProduction: true,
      // highlight-end
    }
  }
}
```

<span id="avoid-concurrent-migration-tasks"></span>

### Evitar Tarefas de Migração Simultâneas

Quando implementamos a nossa aplicação de AdonisJS em produção em vários servidores, temos de certificar-nos de executar as migrações a partir de apenas um servidor e não todos eles.

Para MySQL e PostgreSQL, o Lucid obterá as [fechaduras consultivas](https://www.postgresql.org/docs/9.4/explicit-locking.html#ADVISORY-LOCKS) para assegurar que a migração simultânea não é permitida. No entanto, é muito melhor em primeiro lugar evitar executar migrações a partir de vários servidores.

<span id="persistent-storage-for-file-uploads"></span>

## Armazenamento Persistente para os Carregamentos de Ficheiro

As plataformas de implementação de produção modernas como a Amazon ECS, Heroku, ou aplicações da DigitalOcean executam o código da nossa aplicação dentro um [sistema de ficheiro efémero](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem), o que significa que cada implementação em produção destruirá de maneira nuclear o sistema de ficheiro existente e cria um completamente novo.

Nós perderemos os ficheiros carregados pelo utilizador se estavam armazenados dentro do mesmo espaço que o código da nossa aplicação. Por isto, devemos considerar usar a [Drive](../digging-deeper/drive.md) para guardar os ficheiros carregados pelo utilizador num serviço de armazenamento da nuvem como Amazon S3 ou Google Cloud Storage.

<span id="logging"></span>

## Registo em Diário

O [Registador da AdonisJS](../digging-deeper/logger) escreve registos à `stdout` e `stderr` no formato de JSON. Nós podemos ou configurar um serviço de registo em diário externo para ler os registos a partir da `stdout`/`stderr`, ou enviá-los para um ficheiro local no mesmo servidor.

O núcleo da abstração ou pacotes do ecossistema escrevem os registos no nível da `trace`. Portanto, devemos definir o nível de registo em diário ao `trace` quando depuramos os recursos internos da abstração.

<span id="debugging-database-queries"></span>

## Depurando as Consultas da Base de Dados

O delineador relacional de objeto Lucid emite o evento `db:query` quando a depuração da base de dados está ligada. Nós podemos ouvir este evento e depurar as consultas de SQL usando o registador.

Segue-se um exemplo de impressão bonita das consultas de base de dados em desenvolvimento e usando um registador em produção:

```ts
// title: start/event.ts
import Event from '@ioc:Adonis/Core/Event'
import Logger from '@ioc:Adonis/Core/Logger'
import Database from '@ioc:Adonis/Lucid/Database'
import Application from '@ioc:Adonis/Core/Application'

Event.on('db:query', (query) => {
  if (Application.inProduction) {
    Logger.debug(query.sql)
  } else {
    Database.prettyPrint(query)
  }
})
```

<span id="environment-variables"></span>

## Variáveis de Ambiente

Nós devemos manter as nossas variáveis de ambiente de produção seguras e não mantê-las ao lado do código da nossa aplicação. Se estivermos a usar uma plataforma de implementação de produção como a Cleavr, Heroku, e assim por diante, devemos gerir as variáveis de ambiente a partir do seu painel de controlo.

Quando implementamos o nosso código em produção sobre um servidor básico, podemos manter as nossas variáveis de ambiente dentro do ficheiro `.env`. O ficheiro também pode morar fora da base de código da aplicação. Devemos certificar-nos de informar a AdonisJS sobre a sua localização usando a variável de ambiente `ENV_PATH`:

```sh
cd build

ENV_PATH=/etc/myapp/.env node server.js
```

<span id="caching-views"></span>

## Armazenamento das Visões para Consulta Imediata

Nós devemos armazenar os modelos de marcação do Edge para consulta imediata em produção usando a variável de ambiente `CACHE_VIEWS`. Os modelos de marcação são armazenados na memória em tempo de execução, e nenhum pré-compilação é exigida:

```dotenv
CACHE_VIEWS=true
```

<span id="serving-static-assets"></span>

## Servindo Recursos Estáticos

Servir os recursos estáticos efetivamente é essencial para o desempenho da nossa aplicação. Apesar de quão rápido as nossas aplicações de AdonisJS são, a entrega dos recursos estáticos desempenha um grande papel para uma melhor experiência de utilizador.

<span id="using-a-cdn-service"></span>

### Usando um Serviço de Rede de Entrega de Conteúdo

A melhor abordagem é usar uma rede de entrega de conteúdo para entregar os recursos estáticos a partir da nossa aplicação de AdonisJS.

Os recursos estáticos do front-end compilados usando a [Webpack Encore](../http/assets-manager.md) são assinados com uma impressão digital e isto permite o nosso servidor da rede de entrega de conteúdo armazená-los agressivamente para consulta imediata.

Dependendo do serviço de rede de entrega de conteúdo que estivermos a usar e da nossa técnica de implementação de produção, podemos ter de adicionar uma etapa ao nosso processo de implementação da produção para mover os ficheiros estáticos para o serviço de rede de entrega de conteúdo. Isto é como deveria funcionar uma casca de noz.

- Atualizar `webpack.config.js` para usar a URL da rede de entrega de conteúdo quando criamos a construção de produção:

  ```js
  if (Encore.isProduction()) {
    Encore.setPublicPath('https://your-cdn-server-url/assets')
    Encore.setManifestKeyPrefix('assets/')
  } else {
    Encore.setPublicPath('/assets')
  }
  ```
- Construir a nossa aplicação de AdonisJS como habitual.
- Copiar a saída de `public/assets` para o nosso servidor de rede de entrega de conteúdo. Por exemplo, [eis um comando](https://github.com/adonisjs-community/polls-app/blob/main/commands/PublishAssets.ts)  que usamos para publicar os recursos à um balde da Amazon S3.

---

<span id="using-nginx-to-deliver-static-files"></span>

### Usando o NGINX para entregar ficheiros estáticos

Uma outra opção é descarregar a tarefa de servidor os recursos ao NGINX. Se usamos a Webpack Encore para compilar os recursos do front-end, devemos armazenar agressivamente todos os ficheiros estáticos para consulta imediata uma vez que são assinados com impressão digital.

Adicionar o seguinte bloco ao nosso ficheiro de configuração do NGINX. **Devemos certificar-nos de substituir os valores dentro os parênteses angulares `<>`**:

```nginx
location ~ \.(jpg|png|css|js|gif|ico|woff|woff2) {
  root <PATH_TO_ADONISJS_APP_PUBLIC_DIRECTORY>;
  sendfile on;
  sendfile_max_chunk 2mb;
  add_header Cache-Control "public";
  expires 365d;
}
```

---

<span id="using-adonisjs-static-file.server"></span>

### Usando Servidor de Ficheiro Estático da AdonisJS

Nós também podemos depender do servidor de ficheiro estático embutido da AdonisJS para servir os ficheiros estáticos a partir do diretório `public` para manter as coisas simples.

Nenhum configuração adicional é exigida. Só precisamos de implementar a nossa aplicação de AdonisJS em produção como habitual, e requisitar os ficheiros estáticos que serão servidos automaticamente. No entanto, se usamos a Webpack Encore para compilar os nossos recursos de front-end, devemos atualizar o ficheiro `config/static.ts` com as seguintes opções:

```ts
// title: config/static.ts
{
  // ... resto da configuração
  maxAge: '365d',
  immutable: true,
}
```
