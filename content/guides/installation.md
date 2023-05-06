---
summary: Começar pela Criação e Execução de uma Aplicação de AdonisJS
---

A AdonisJS é uma abstração de Node.js, e por isto exige que Node.js esteja instalada no teu computador. Para ser mais preciso, precisamos de pelo menos do lançamento mais recente da `Node.js v14`.

Tu podes consultar as versões da Node.js e npm executando os seguintes comandos:

```sh
# consulta versão da node.js
node -v
```

Se não tiveres a Node.js instalada, podes [descarregar o binário](https://nodejs.org/en/download) para o teu sistema operativo a partir da página oficial.

Se estiveres confortável com a linha de comando, então recomendamos usar [Volta](https://volta.sh) ou [Gestor de Versão de Node](https://github.com/nvm-sh/nvm) para instalar e executares várias versões de Node.js no teu computador.

## Criando um Novo Projeto

Tu podes criar um novo projeto usando [`npm init`](https://docs.npmjs.com/cli/v7/commands/npm-init), [yarn create](https://classic.yarnpkg.com/en/docs/cli/create) ou [`pnpm create`](https://pnpm.io/tr/next/cli/create). Estas ferramentas descarregarão o pacote de arranque da AdonisJS e iniciar o processo de instalação:

:::codegroup

```sh
// title: npm
npm init adonis-ts-app@latest hello-world
```

```sh
// title: yarn
yarn create adonis-ts-app hello-world
```

```sh
// title: pnpm
pnpm create adonis-ts-app hello-world
```
:::

O processo de instalação serve de ponto para as seguintes escolhas:

#### Estrutura do Projeto

Tu podes escolher entre uma das seguintes estruturas de projeto:

- `web`: estrutura de projeto ideal para a criação de aplicações clássicas geradas pelo servidor. Nós configuramos o suporte para sessões e também instalamos o motor de modelo de marcação da AdonisJS.
- `api`: estrutura de projeto ideal para criação dum servidor de API.
- `slim`: estrutura de projeto ideal para criar a aplicação de AdonisJS mais minimalista possível e que não instala nenhum pacote adicional, exceto o núcleo da abstração.

---

#### Nome do Projeto

O nome do projeto. Nós definimos o valor para este dentro do ficheiro `package.json`.

---

#### Configurar o ESLint/Prettier

Opcionalmente, podes configurar a eslint e prettier. Ambos os pacotes são configurados com definições opiniosas usadas pela equipa principal da AdonisJS.

---

#### Configurar a Webpack Encore

Opcionalmente, podes também configurar a [Webpack Encore](./http/assets-manager.md) para empacotares e servir dependências de frontend.

> Nota: a AdonisJS é uma abstração de backend e não tem como prioridade lidar com recursos de frontend. Por isso a configuração da Webpack é opcional.

## Iniciando o Servidor de Desenvolvimento

Depois de criares a aplicação, podes iniciar o servidor de desenvolvimento usando o seguinte comando:

```sh
node ace serve --watch
```

- O comando `serve` inicia o servidor de HTTP e realiza uma compilação em memória da TypScript para JavaScript.
- A opção `--watch` liga a observação de mudanças no sistema de ficheiro e reiniciar o servidor automaticamente sempre que houver alguma.

Por padrão, o servidor começa no porta `3333` (definida dentro do ficheiro `.env`). Tu podes visualizar a página de boas-vindas ao visitar: http://localhost:3333.

## Compilando para Produção

Tu deves sempre servir em produção o JavaScript compilado. Tu podes criar uma construção de produção executando o seguinte comando:

```sh
node ace build --production
```

A saída compilada é colocada na pasta `build`. Tu podes entrar para dentro desta pasta e iniciar o servidor executando diretamente o ficheiro `server.js`. Saiba mais sobre o [processo de construção da TypeScript]:

```sh
cd build
node server.js
```
