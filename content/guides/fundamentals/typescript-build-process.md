---
summary: Saiba como a AdonisJS constrói a nossa aplicação de TypeScript em ambos desenvolvimento e produção. 
---

Um dos objetivos da abstração é fornecer suporte de primeira classe para TypeScript. Este vai além dos tipos estáticos e sensor inteligente que podemos desfrutar enquanto escrevemos o código.

**Nós também garantimos que nunca tenhamos de instalar quaisquer ferramentas de construção adicional para compilar o nosso código durante o desenvolvimento ou produção.**

:::note
Este guia assume que temos algum conhecimento sobre a TypeScript e o ecossistema de ferramentas de construção.
:::

<span id="common-bundling-approaches"></span>

## Abordagens de Construção Comuns

A seguir estão algumas das abordagens comuns para o desenvolvimento duma aplicação de Node.js escrita em TypeScript.

<span id="using-tsc"></span>

### Usando a `tsc`

A maneira mais simples de compilar o nosso código de TypeScript para JavaScript é usando a linha de comando `tsc` oficial.

- Durante o desenvolvimento, podemos compilar o nosso código no modo de observação usando o comando `tsc --watch`.
- Depois, podemos agarrar a `nodemon` para observar a saída compilada (código de JavaScript) e reiniciar o servidor de HTTP sobre toda mudança. Por este tempo, temos dois observadores em execução.
- Além disto, podemos ter de escrever [programas personalizados para copiar ficheiros estáticos](https://github.com/microsoft/TypeScript/issues/30835) como **modelos de marcação** para a pasta de construção para que o nosso código de JavaScript em execução consiga encontrá-lo e referenciá-lo.

---

<span id="using-ts-node"></span>

### Usando a `ts-node`

`ts-node` melhora a experiência de programação, uma vez que compila o código em memória e não escreve-o sobre o disco. Assim, podemos combinar a `ts-node` e `nodemon` e executar no nosso código de TypeScript como um cidadão de primeira classe.

No entanto, para aplicações maiores, a `ts-node` pode tornar-se lenta uma vez que tem de recompilar o projeto inteiro sobre toda mudança de ficheiro. Em contraste, a `tsc` estava apenas a reconstruir o ficheiro mudado.

Nota que, a `ts-node` é uma ferramenta apenas de desenvolvimento. Então ainda temos de compilar o nosso código para JavaScript usando `tsc` e escrever programas personalizados para copiar os ficheiros estáticos para produção.

---

<span id="using-webpack"></span>

### Usando a Webpack

Depois de tentar as abordagens acima, podemos decidir dar à Webpack uma change. A Webpack é uma ferramenta de construção e tem muito à oferecer. Mas, vem com suas próprias desvantagens:

- Primeiramente, usar a Webpack para empacotar o código do backend é um exagero. Nós talvez nem precisemos de 90% das funcionalidades da Webpack criadas para servir o ecossistema de frontend.
- Nós podemos ter de repetir algumas configurações na configuração do `webpack.config.js` e no ficheiro `tsconfig.json` principalmente, quais ficheiros observar e ignorar.
- Além disto, nem temos a certeza se podemos instruir a [Webpack a NÃO empacotar](https://stackoverflow.com/questions/40096470/get-webpack-not-to-bundle-files) o backend inteiro num único ficheiro.

<span id="adonisjs-approach"></span>

## Abordagem da AdonisJS

Nós não somos grandes fãs de ferramentas de construção muito complicadas e compiladores de ponta. Ter uma experiência de programação é muito mais valioso do que expor a configuração para afinar cada botão.

Nós começamos com os seguintes conjuntos de objetivos:

- Manter o compilador oficial da TypeScript e não usar quaisquer outras ferramentas como `esbuild` ou `sw`. Elas são excelentes alternativas, mas não suportam algumas das funcionalidades da TypeScript (por exemplo, [a API de Transformadores](https://levelup.gitconnected.com/writing-typescript-custom-ast-transformer-part-1-7585d6916819)).
- O ficheiro `tsconfig.json` existente deve lidar com todas as configurações.
- Se o código executar em desenvolvimento, então também deve executar em produção. O que significa que, não usa duas ferramentas de desenvolvimento e produção completamente diferente e depois ensinar as pessoas como ajustar o seu código.
- Adicionar suporte leve para copiar ficheiros estáticos para a pasta de construção final. Normalmente, estes serão os modelos de marcação do Edge.
- **Certificar que o REPL também pode executar o código da TypeScript como cidadão de primeira classe. Todas as abordagens acima, exceto `ts-node`, não conseguem compilar e avaliar o código de TypeScript diretamente.**

<span id="in-memory-development-compiler"></span>

## Compilador de Desenvolvimento em Memória

Semelhante à `ts-node`, críamos o módulo [`@adonisjs/require-ts`](https://github.com/adonisjs/require-ts). Ele usa a API de compilador da TypeScript, significa que todas as funcionalidades da TypeScript funciona, e o teu ficheiro `tsconfig.json` é a única fonte de verdade.

No entanto, `@adonisjs/require-ts` é ligeiramente diferente da `ts-node` das seguintes maneiras.

- Nós não realizamos nenhuma verificação de tipo durante o desenvolvimento e exceto confiar no nosso editor de código para o mesmo.
- Nós armazenamos a [saída compilada](https://github.com/adonisjs/require-ts/blob/develop/src/Compiler/index.ts#L185-L223) numa pasta de armazenamento de consulta imediata. Então da próxima vez que o nosso servidor reiniciar, não precisamos de recompilar os ficheiros não modificados. Isto melhora a velocidade dramaticamente.
- Os ficheiros armazenados para consulta imediata têm de ser eliminados em algum ponto. O módulo `@adonisjs/require-ts` expõe os [métodos auxiliares](https://github.com/adonisjs/require-ts/blob/develop/index.ts#L43-L57) que o observador de ficheiro da AdonisJS usa para limpar o armazenamento de consulta imediata para o ficheiro modificado recentemente.
- Limpar o armazenamento de consulta imediata é apenas essencial para reivindicar o espaço do disco. Não impacta o comportamento do programa.

Toda vez que executarmos `node ace serve --watch`, iniciaremos o servidor de HTTP juntamente com o compilador em memória e observaremos o sistema de ficheiro por mudanças de ficheiro.

<span id="standalone-production-builds"></span>

## Construções de Produção Autónomas

Nós construímos o nosso código para produção executando o comando `node ace build --production`. Ele realiza as seguintes operações:

- Limpar o diretório `build` existente (se existir algum).
- Construir os nossos recursos do frontend usando a Webpack Encore (apenas se estiver instalada).
- Usar a API do compilador da TypeScript para compilar o código da TypeScript para JavaScript e escrevê-lo dentro da pasta `build`. **Desta vez, realizamos a verificação de tipo e reportamos os erros da TypeScript**.
- Copiar todos os ficheiros estáticos para a pasta `build`. Os ficheiros estáticos são registados dentro do ficheiro `adonisrc.json` sob o vetor `metaFiles`.
- Copiar o `package.json` e `package-lock.json` ou `yarn.lock` para a pasta `build`.
- Gerar o ficheiro `ace-manifest.json`. Ele contém um índice de todos os comandos que o nosso projeto está usar.
- É tudo.

---

<span id="why-do-we-call-it-a-standalone-build"></span>

#### Porquê a Chamamos uma Construção Autónoma?

Depois de executar o comando `build`, a pasta de saída tem tudo que precisamos para implementar a nossa aplicação em produção.

Nós podemos copiar a pasta `build` sem o nosso código-fonte de TypeScript, e a nossa aplicação funcionará muito bem.

Criar uma pasta `build` autónoma ajuda na redução do tamanho do código que implementamos no nosso servidor de produção. Isto é normalmente útil quando empacotamos a nossa aplicação como uma imagem de Docker. No entanto, não existe necessidade de ter ambas a fonte e a saída da construção na nossa imagem de Docker e mantê-la leve.

---

<span id="points-to-keep-in-mind"></span>

#### Pontos à Manter em Mente

- Pós-construção, a pasta de saída torna-se a raiz da nossa aplicação de JavaScript.
- Nós devemos sempre entrar dentro da pasta `build` usando `cd` e depois executar a nossa aplicação:

  ```sh
  cd build
  node server.js
  ```

- Nós devemos instalar as dependências exclusivas de produção dentro da pasta `build`:

  ```sh
  cd build
  npm ci --production
  ```

- Nós não copiamos o ficheiro `.env` para a pasta de saída. Uma vez que as variáveis de ambiente não são transferíveis, devemos definir as variáveis de ambiente para a produção separadamente.
