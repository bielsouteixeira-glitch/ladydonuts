# Lady Donuts — publicação (Vercel) e manutenção

## Stack identificada

- **Vite 7 + React 19 + TypeScript + Tailwind CSS 4** (single-page, sem rotas de cliente).
- Build: `npm run build` → saída em **`dist/`** (`index.html` com JS/CSS embutidos via
  `vite-plugin-singlefile`, mais `dist/images/*.svg` e `robots.txt`).
- Requer Node.js **20.19+ ou 22+** (exigência do Vite 7). As versões padrão da Vercel
  (20.x / 22.x) atendem. Não há variáveis de ambiente.

## Publicar na Vercel

As configurações já estão fixadas em `vercel.json` (não dependem do painel):

| Configuração     | Valor            |
| ---------------- | ---------------- |
| Framework Preset | Vite             |
| Build Command    | `npm run build`  |
| Output Directory | `dist`           |
| Install Command  | padrão (`npm install`, usa o `package-lock.json`) |

**Opção A — via Git:** suba o projeto para GitHub/GitLab/Bitbucket → Vercel → *Add New Project*
→ *Import* → *Deploy*. Root Directory deve ser a raiz deste projeto (onde está o `package.json`).

**Opção B — via CLI:** na pasta do projeto, `npx vercel` (preview) ou `npx vercel --prod`.

> Se o projeto já existia na Vercel com *Framework Preset: Other* e sem Output Directory,
> a Vercel publicava a pasta `public/` ou o `index.html` sem compilar (página em branco /
> 404). O `vercel.json` corrige isso. Se o painel tiver valores manuais diferentes,
> ajuste-os para os da tabela acima ou marque *Override* desligado.

### Checklist após o deploy

1. Abrir a URL gerada: hero, "Nossos donuts e cookies", "Sobre" e Instagram exibem as fotos.
2. Menu (desktop e mobile), âncoras Início/Produtos/Sobre/Pedidos, lightbox das fotos.
3. Botões do Instagram abrem `https://www.instagram.com/donuts_lady/` em nova aba.
4. No DevTools → Network, as fotos vêm de `drive.google.com/thumbnail?...` (fonte 1).
   Cada `<img>` tem `data-source="1|2|3"` indicando qual fonte foi usada.

## Fotografias (Google Drive) — como funciona

Ordem de tentativa de cada foto (`src/components/DriveImage.tsx` + `imageSources` em
`src/data/site.ts`):

1. `https://drive.google.com/thumbnail?id=ID&sz=w1600` — **método prioritário**
2. `https://lh3.googleusercontent.com/d/ID=w1600` — CDN para onde o thumbnail redireciona
3. `/images/<nome>.svg` — cópia local **da mesma foto real** (1000px), incluída no build
4. Aviso visível "Não foi possível carregar esta foto" (falhas nunca são escondidas)

Avança para a próxima fonte em caso de erro ou se a fonte remota não responder em 12 s.

IDs originais (não alterar): ver `photos` em `src/data/site.ts`.
Para regenerar as cópias locais: `node scripts/make-fallbacks.mjs`.

Para adicionar uma foto nova: compartilhe o arquivo como "qualquer pessoa com o link",
copie o ID de `https://drive.google.com/file/d/ID/view` e inclua um item em `photos`.

## Pendências que dependem da marca

| Item | Onde ajustar | Situação |
| --- | --- | --- |
| Links da loja no iFood, 99Food e Keeta | `deliveryChannels[].url` em `src/data/site.ts` | **Não confirmados** — os cartões mostram "Busque por Lady Donuts no app" até o link ser informado |
| Logo original | `brand.logoSrc` em `src/data/site.ts` (ex.: `/images/logo.svg`) | **Arquivo não fornecido** — o site usa marca tipográfica |
| Fotos de cookies / vídeos | `photos` em `src/data/site.ts` | Não havia nos materiais recebidos |

Assim que um link de delivery for preenchido, o botão "Pedir no …" aparece
automaticamente na seção Pedidos e no rodapé.

## Netlify (alternativa)

`netlify.toml` continua no projeto (build `npm run build`, publish `dist`) e não interfere
na Vercel.
