# Villa Lobos Office Park — site estático

Recriação do site institucional (hoje WordPress + Elementor + Directorist) como site
estático em Astro + Tailwind, para deploy no Cloudflare Pages.

Referência original: https://villalobosofficepark.com.br/

## Design tokens

| Token | Valor | Uso | Original extraído |
|---|---|---|---|
| `ink` | `#3E4C59` | Cor de texto padrão | igual |
| `mist` | `#F5F5F5` | Fundo de seção | igual |
| `fog` | `#F9F9FB` | Fundo de seção alternado + footer | igual |
| `haze` | `#E9E9E9` | Fundo do wrapper do site | igual |
| `accent` | `#A1B9CE` | Badges, destaques | igual |
| `sand` | `#746145` | Eyebrows + fundo dos blocos de números | `#C2B6A2` |
| `mint` | `#4C6B58` | Botões (bg) + links do footer (text) | `#A3C2B3` |
| `sage` | `#4F6E62` | Item ativo do menu | `#82A498` |
| `graytext` | `#5C5C5C` | Parágrafos de corpo | `#7A7A7A` |
| `carbon` | `#404040` | Texto dos cards de empresa | igual |
| `sand-claro` | `#C2B6A2` | Só ícones decorativos da barra de categorias | igual |
| `sand-bloco` | `#9C8B6E` | Fundo dos blocos de números (texto branco) | `#C2B6A2` |

`sand` tem três variantes porque o mesmo bege do original é usado em papéis com exigências
de contraste diferentes: como **texto** sobre fundo claro (precisa ser escuro, `#746145`),
como **fundo** atrás de texto branco grande (`#9C8B6E`, dá 3.32:1 — acima do mínimo de 3:1
para texto ≥24px) e como **ícone decorativo**, onde não há regra de contraste e o tom
original `#C2B6A2` é mantido.

**`sand`, `mint`, `sage` e `graytext` foram escurecidos** em relação ao valor extraído do site
original: o valor original falhava WCAG AA (Lighthouse Acessibilidade 93, 24 ocorrências
de contraste insuficiente — texto cinza claro e botões brancos sobre verde-sálvia claro
demais). Mantém a mesma paleta e reconhecibilidade visual, mas passa em 100 no Lighthouse.
Não reverter para os valores originais sem também resolver o contraste de outra forma.

Fontes: **Roboto** (corpo, `font-sans`) e **Manrope** (títulos, `font-display`), via Google Fonts.

Referência de layout: Home tem 7425px de altura a 1536px de largura, header 169px, footer 529px.

## Dados das empresas

`src/data/empresas.json` guarda as 75 empresas (nome, slug, categorias, contato, sala,
prédio, horário, descrição) e a lista das 19 categorias. Substitui o plugin Directorist.
Extraído das 5 páginas paginadas do original + as 75 páginas de detalhe.

`/empresas` renderiza **todas as 75 de uma vez** com filtro de categoria client-side, em vez
da paginação de 15 por página do original — escolha deliberada, já que num site estático
paginar exigiria 5 rotas e recarregamentos. O visual do card é fiel ao original
(344×260 de imagem, radius 12px, grid de 3 colunas com gap 30px).

Logos em `public/img/empresas/<slug>.webp`, redimensionados para no máximo 600px de largura
(os originais do WordPress somavam 43,6 MB; em WebP somam 0,48 MB). Ao adicionar uma empresa
nova, passar o logo pela mesma conversão.

## Formulário de contato

`/contato` tem o formulário do original (Nome, Sobrenome, Telefone, E-mail, Empresa,
Mensagem), mas **sem backend**: o submit monta um `mailto:` para
`assistente.administrativo@cvlop.com.br`. Funciona sem infra, mas depende do cliente de
e-mail do visitante. Solução definitiva pendente de decisão (Cloudflare Pages Function,
Formspree ou Supabase).

## Embeds de terceiros

O vídeo institucional em `/sobre-nos` usa **facade**: mostra `/img/video-capa.jpg` e só
injeta o iframe do YouTube no clique. Sem isso o Lighthouse reprova em Best Practices
(cookies de terceiros) e a página carrega ~1 MB de JS do player à toa. Manter esse padrão
em qualquer embed novo.

## Animações

O original usa animações do Elementor (animate.css) disparadas ao entrar na viewport.
Recriadas com CSS + IntersectionObserver, sem biblioteca:

- `data-reveal="fadeIn|fadeInUp|fadeInLeft|fadeInRight|slideInUp|slideInRight|zoomIn"`
  + `data-reveal-delay="600"` (ms)
- As variantes `slide*` **não** usam fade: mantêm opacidade 1 e só deslizam, como no original
- O observador em `Base.astro` adiciona a classe `.revelado`; o CSS mora em `global.css`
- `data-contador="3" data-contador-duracao="2000"` faz a contagem animada (o "3" de eventos)
- Tudo respeita `prefers-reduced-motion`

Os deslocamentos usam distância fixa (45–60px) em vez do `-100%` do animate.css, que
causaria overflow horizontal. Os delays por elemento seguem os do original.

## Foto fixa no scroll (profundidade)

Quatro seções do original usam `background-attachment: fixed` — a foto fica parada
enquanto a seção rola por cima, criando profundidade. Classe `.foto-fixa` em `global.css`:

| Página | Seção | `background-position` |
|---|---|---|
| Home | CTA "INTERESSADO EM FAZER PARTE…" | `50% 100%` |
| Sobre Nós | Banner com o logo claro | `50% 0%` |
| Sobre Nós | CTA "CONHEÇA NOSSOS ESPAÇOS…" | `50% 50%` |
| Contato | Hero "FALE CONOSCO" | `50% 50%` |

O efeito só liga a partir de `lg`. **iOS Safari ignora `background-attachment: fixed`
combinado com `background-size: cover`** e renderiza a foto esticada e fora de lugar, então
abaixo de 1024px fica `scroll` (a foto simplesmente rola junto, sem quebrar nada). Também
desliga com `prefers-reduced-motion`.

## Regra de fidelidade visual

Ao recriar qualquer seção, **medir o original, não estimar a partir de screenshot**: rodar
`getComputedStyle` via chrome-devtools MCP no elemento correspondente do site ao vivo e usar
os valores exatos. Screenshots servem só para confirmação final, sempre salvos com `filePath`
e capturados por seção — nunca full-page inline.

Breakpoints de validação: 1536px, 1024px, 390px.

## Deploy

Ao vivo em **https://villalobos-office-park.pages.dev** (Cloudflare Pages, conta
Juanworksspace@gmail.com). Publicar com `npm run deploy`.

⚠️ **O `wrangler pages deploy` tenta migrar o projeto para Workers automaticamente**: ele
roda `astro add cloudflare`, instala o adapter `@astrojs/cloudflare`, cria `wrangler.jsonc`
e adiciona bindings de KV/Images que este site estático não usa. O projeto Pages já foi
criado com `--force`, então os deploys seguintes vão direto para o Pages. Se a delegação
voltar a acontecer, reverter: remover o adapter do `astro.config.mjs` e do `package.json`,
apagar `wrangler.jsonc` e `.wrangler/deploy/`.

Este site é estático e **não deve** ter adapter, KV ou Images.

⚠️ Enquanto o contrato terceirizado estiver ativo, **não apontar o DNS de
`villalobosofficepark.com.br`** para cá. A URL `.pages.dev` é a demo.

## Assets

O vídeo do hero foi recomprimido de 44,6 MB para 11,2 MB (H.264, CRF 27, sem áudio,
`+faststart`) porque **o Cloudflare Pages recusa arquivos acima de 25 MB**. Qualquer
reenvio do vídeo original precisa passar pela mesma compressão. VP9/WebM foi testado e
saiu maior que o H.264 — não vale a pena aqui.

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
