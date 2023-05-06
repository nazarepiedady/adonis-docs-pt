A AdonisJS é uma coleção de vários pacotes de terceiros construídos em torno do [núcleo da abstração](https://github.com/adonisjs/core). Sempre que ouvires-nos mencionar a versão da AdonisJS, presuma que estamos a falar sobre a versão do núcleo da abstração.

Todos os outros pacotes como `@adonisjs/lucid`, ou `@adonisjs/mail` têm suas próprias versões independentes e são livres de ter seu ciclo de lançamento.

## Seguindo o Versionamento Semântico

Nós seguimos estritamente o [versionamento semântico](https://semver.org/) e batemos a versão principal depois de todas mudanças de rutura. Isto significa que, o que é AdonisJS 5 hoje pode rapidamente tornar-se AdonisJS 8 em alguns meses.

- Nós bateremos a versão de remendo quando lançarmos as **correções de erro de programação críticos** (por exemplo: 5.2.0 para 5.2.1).
- A versão secundária inclui **novas funcionalidades** ou **correções de erro de programação que não é crítico**. Além disto, depreciaremos as APIs durante um lançamento secundário. (por exemplo: 5.2.0 para 5.3.0)
- Quando lançamos mudanças de rutura, batemos a versão principal (por exemplo: 5.2.0 para 6.0.0).

## Introduzindo Mudanças de Rutura

A medida que a AdonisJS for amadurecendo, assumiremos mais responsabilidade de não introduzir mudanças de rutura de quando em quando, e todas as mudanças de rutura **deveria passar por uma fase de depreciação e RFC**.

Antes de introduzirmos qualquer mudança de rutura, publicaremos um [RFC](https://github.com/adonisjs/rfcs) discutindo as motivações por trás da mudança. Se não houver um adiamento significativo, avançaremos com a mudança.

A fase inicial da mudança depreciará as APIs existentes durante um lançamento secundário. Se executares a tua aplicação depois desta mudança receberás muitos avisos, mas não quebrará nada e continuará a funcionar como está.

Depois de uma fase de resfriamento dum mínimo de 4 semanas, removeremos as APIs depreciadas durante o próximo lançamento principal. A remoção de código antigo ou morto é essencial para assegurar que base de código da abstração esteja bem mantida e não empanturrada com variações passadas.

As seguintes mudanças não estão sujeitas às mudanças de rutura:

- **APIs não documentadas e estruturas de dados internas** podem ser mudadas durante qualquer lançamento. Então, se dependeres de APIs não documentadas ou membros de classe privadas, estás por conta própria quando mudarmos ou as reestruturarmos.
- **Alfa e próxima versões da AdonisJS** podem receber mudanças de rutura sem uma batida de versão principal. Isto porque queremos a liberdade criativa para iterar rapidamente com base nos nossos aprendizados no período alfa.

## Ciclo de Lançamento

A AdonisJS segue rigorosamente um ciclo de lançamento de 8 semanas para entregar novas funcionalidades ou publicar mudanças de rutura. No entanto, correções de erro de programação e remendos de segurança são normalmente lançados de imediato.

Tu podes consultar o nosso [mapa de estradas na Trello](https://trello.com/b/3klaHbfP/adonisjs-roadmap) e [o que está no próximo cartão de lançamento](https://trello.com/c/y8PCAodY/47-september-planning-2021) para saberes sobre as próximas mudanças.
