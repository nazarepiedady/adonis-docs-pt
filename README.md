<div align="center">
  <img src="https://res.cloudinary.com/adonisjs/image/upload/q_100/v1558612869/adonis-readme_zscycu.jpg" width="600px">
</div>

<br />

<div align="center">
  <h3>Documentação Oficial da AdonisJS em Português</h3>
  <p>
    Código-fonte e documentação para página oficial da documentação hospedado na <a href="https://docs.adonisjs.com">docs.adonisjs.com</a>
  </p>
</div>

## Configuração

Siga os passos mencionados abaixo para configurares o teu projeto local:

- Bifurque o repositório
- Empurre o repositório no teu local
- Instale todas as dependências usando `npm install`.
- Inicie o servidor de desenvolvimento da AdonisJS usando `node ace serve --watch`

**Nós não seguimos nenhum processo para transformar documento markdown em HTML**. No lugar disto, compilamos os ficheiros em cada requisição de página. Isto garante que não tenhamos de executar nenhum compilador de fundo para compilar o markdown e depois recompilar tudo numa única mudança. O processo é tão simples quanto:

```
--> Nova requisição de HTTP --> Encontrar o ficheiro markdown para a URL --> Compilá-lo e servi-lo
```

## Variáveis de Ambiente

As seguintes variáveis de ambiente são obrigatórias para iniciar o servidor de desenvolvimento ou criar uma construção de produção:

```
PORT=3333
HOST=0.0.0.0
NODE_ENV=development
APP_KEY=iPbHJ0Wdr8_hA4DLTj83lKedQP9I5rJO
CACHE_VIEWS=false
DEBUG_DOCS=true
ALGOLIA_API_KEY=
COPY_REDIRECTS_FILE=false
```

A variável de ambiente `ALGOLIA_API_KEY` é opcional e exigida apenas se quiseres ativar a pesquisa de conteúdo.

Se estiveres a implementar em produção uma versão traduzida da documentação, então defina a `COPY_REDIRECTS_FILE=false`. Já que o ficheiro de redirecionamentos é apenas aplicável para a documentação oficial para evitar quebrar as URLs de `previw.adonisjs.com`.

## Estrutura do Conteúdo

O conteúdo de markdown é guardado dentro do diretório `content` e cada tipo de documentação (nós as chamamos de zonas) tem sua própria sub-pasta.

A navegação para cabeçalho da página e barra lateral é orientada pelo ficheiro `menu.json` dentro de cada sub-diretório da zona:

```
content
├── cookbooks
│   ├── menu.json
├── guides
│   ├── menu.json
├── reference
│   └── menu.json
└── releases
    ├── menu.json
```

O ficheiro `menu.json` tem a seguinte estrutura:

```json
{
  "name": "Database",
  "categories": [
    {
      "name": "root",
      "docs": [
        {
          "title": "Connection",
          "permalink": "database/connection",
          "contentPath": "database/connection.md",
        }
      ]
    }
  ]
}
```

- O objeto de alto nível é nome do grupo. Tu podes ter um ou mais grupos dentro duma zona e serão listados num menu deslizante na navegação do cabeçalho.
- Se não houver nenhum grupo. Tu podes nomeia-lo como `root`.
- Cada grupo pode ter um ou mais `categories`. As categorias são listadas dentro da barra lateral.
- A categoria com nome `root` não será impressa no HTML.
- Cada categoria pode ter um ou mais `docs`.
- A documentação ainda tem um `title`, `permalink` e o caminho para o ficheiro de conteúdo. **O caminho é relativo da zona raiz.**

## Gerando o HTML

Nós fazemos uso dum módulo [@dimerapp/content](https://npm.im/@dimerapp/content) escrito por nós para transformar o markdown em HTML usando modelos de marcação de Edge no meio.

Nós começamos primeiro por converter o Markdown numa Árvore de Sintaxe Abstrata e depois transformamos cada nó Árvore de Sintaxe Abstrata usando os modelos de marcação de Edge. Isto permite separar o markdown personalizado. Consulte o diretório [./resources/views/elements](./resources/views/elements) para veres como estamos a usá-lo.

Os blocos de código são gerados usando o [shiki](https://github.com/shikijs/shiki). O módulo usa a gramática e temas do Visual Studio Code para processar os blocos de código. Além disto, os blocos de código são processados no backend e cliente recebe o HTML formatado.


## Configurando o `@dimerapp/content`

O módulo `@dimerapp/content` não vem com nenhuma interface da linha de comando ou opções sobre como o conteúdo deveria ser estruturado.

Então temos de configurar o módulo `@dimerapp/content`. Isto é feito dentro do ficheiro `start/content.ts`, que depende do ficheiro `config/markdown.ts`.


## CSS e JavaScript de Frontend

Os estilos são escritos em CSS puro e armazenados dentro do diretório `./resources/css`. Para organizarmos um pouco as coisas, temos de separá-los dentro de vários ficheiros `.css`.

Nós também fazemos uso de `alpine.js` para adicionar pequenos componentes interativos, como deslize da navegação de cabeçalho e a alternância do grupo de código.

O `@hotwire/turbo` é usado para navegar entre as páginas sem fazer uma atualização completa.

### JavaScript Personalizado

O redesenho das páginas de HTML reinicia a posição do deslocamento da barra lateral, o que é um pouco irritante. Então fazemos uso de eventos de turbo `turbo:before-render` e `turbo:render` para armazenar a posição da barra lateral e então restaurá-lo depois da navegação.

No primeiro carregamento da página, deslocamos o item da barra lateral ativo para a visão usando o método `scrollIntoView`.

## Implementação em Produção

Execute o seguinte comando para criar a construção de produção.

```sh
npm run build
```

A saída é escrita no diretório `public` e podes servi-lo em qualquer hospedeiro capaz de servir ficheiros estáticos.

A página oficial está hospedada na [pages.cloudflare.com](https://pages.cloudflare.com/)

### Variáveis de Ambientes Importantes

A variável de ambiente `ALGOLIA_API_KEY` é opcional e exigida apenas se quiseres ativar a pesquisa de conteúdo.

Se estiveres a implementar em produção uma versão traduzida da documentação, então defina a `COPY_REDIRECTS_FILE=false`. Já que o ficheiro de redirecionamentos é apenas aplicável para a documentação oficial para evitar quebrar as URLs de `previw.adonisjs.com`.

## Traduzindo a Documentação para Idiomas Diferentes

Tu estás livre de bifurcar este repositório e traduzir a documentação para diferentes idiomas e publicá-los num sub-domínio separado.

> Desmentido: A documentação traduzida é considerada um esforço da comunidade. A página e traduções nunca serão suportadas ou mantidas pela equipa principal.

### Processo de Tradução da Documentação

- Notifique a todos sobre a documentação traduzida nesta [questão fixada](https://github.com/adonisjs/docs.adonisjs.com/issues/1).
- Nós podemos entregar-te um sub-domínio para o idioma uma vez que tiveres traduzido mais de 50% da documentação.
- A medida que a documentação for atualizada na página oficial, deixaremos uma ligação para o pedido de atualização na [questão fixada](https://github.com/adonisjs/docs.adonisjs.com/issues/1) e podes empurrar de volta aquelas mudanças.
- Nós recomendados não localizar as URLs. Apenas traduza o conteúdo da documentação.
- Sinta-se livre para enviar um pedido de atualização para a Algolia para indexar o página traduzida. Cá está uma [referência para a configuração da algolia](https://github.com/algolia/docsearch-configs/blob/master/configs/adonisjs_next.json) da página oficial.
