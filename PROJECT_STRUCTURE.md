# Estrutura do Repositório — Meu PDF

Visão completa da organização de arquivos e pastas do projeto, com a finalidade de cada item.

---

## Raiz do Repositório

```
pdf-tools-suite/
├── README.md
├── CONTRIBUTING.md
├── LICENSE
├── .gitignore
├── .editorconfig
├── src/
└── docs/
```

| Arquivo           | Descrição                                                                                       |
| ----------------- | ----------------------------------------------------------------------------------------------- |
| `README.md`       | Porta de entrada do repositório — visão geral, como rodar localmente, links para a documentação |
| `CONTRIBUTING.md` | Como contribuir: padrões de código, fluxo de PR, como adicionar uma nova ferramenta             |
| `LICENSE`         | Licença MIT                                                                                     |
| `.gitignore`      | `node_modules/`, `.DS_Store`, arquivos de sistema, builds temporários                           |
| `.editorconfig`   | Indentação (2 espaços), charset UTF-8, line endings — consistência entre editores e devs        |

---

## `src/` — Aplicação (Deploy)

Tudo que vai para produção. Pode ser servido diretamente como site estático em GitHub Pages, Netlify ou Vercel sem nenhum build step.

```
src/
├── index.html
├── juntar-pdf.html
├── dividir-pdf.html
├── comprimir-pdf.html
├── word-pdf.html
├── pdf-para-word.html
├── jpg-pdf.html
├── pdf-para-jpg.html
├── excel-pdf.html
├── pdf-para-excel.html
├── pptx-pdf.html
├── pdf-para-pptx.html
├── ocr-pdf.html
├── assinar-pdf.html
├── marca-dagua.html
├── numerar-paginas.html
├── manifest.json
├── robots.txt
├── sitemap.xml
└── assets/
    ├── css/
    │   ├── global.css
    │   ├── themes.css
    │   └── components.css
    ├── js/
    │   ├── utils.js
    │   ├── file-validator.js
    │   └── theme-toggle.js
    ├── icons/
    │   ├── favicon.ico
    │   ├── icon-192.png
    │   ├── icon-512.png
    │   └── icon-512-maskable.png
    └── og/
```

### Páginas

Cada arquivo HTML é uma ferramenta independente e autossuficiente. Não há roteamento central nem estado compartilhado entre páginas.

| Arquivo                | Ferramenta                                               |
| ---------------------- | -------------------------------------------------------- |
| `index.html`           | Home — apresentação, privacy banner, menu de ferramentas |
| `juntar-pdf.html`      | Mesclar múltiplos PDFs em um único arquivo               |
| `dividir-pdf.html`     | Extrair páginas selecionadas de um PDF                   |
| `comprimir-pdf.html`   | Reduzir o tamanho de um arquivo PDF                      |
| `word-pdf.html`        | Converter `.docx` / `.doc` em PDF                        |
| `pdf-para-word.html`   | Converter PDF em documento `.docx`                       |
| `jpg-pdf.html`         | Converter imagens JPG/PNG em PDF                         |
| `pdf-para-jpg.html`    | Extrair páginas de PDF como imagens                      |
| `excel-pdf.html`       | Converter planilha `.xlsx` em PDF                        |
| `pdf-para-excel.html`  | Extrair tabelas de PDF para `.xlsx`                      |
| `pptx-pdf.html`        | Converter apresentação `.pptx` em PDF                    |
| `pdf-para-pptx.html`   | Converter PDF em apresentação PowerPoint                 |
| `ocr-pdf.html`         | Reconhecimento de texto em PDFs digitalizados            |
| `assinar-pdf.html`     | Adicionar assinatura digital em PDF                      |
| `marca-dagua.html`     | Adicionar marca d'água em PDFs                           |
| `numerar-paginas.html` | Adicionar numeração de páginas em PDF                    |

### Assets

#### `assets/css/`

| Arquivo          | Conteúdo                                                                    |
| ---------------- | --------------------------------------------------------------------------- |
| `global.css`     | Reset, tipografia, layout base, utilitários                                 |
| `themes.css`     | Custom properties do Catppuccin — Latte (claro) e Mocha (escuro)            |
| `components.css` | Componentes reutilizáveis: dropzone, botões, progress bar, toast, file card |

#### `assets/js/`

| Arquivo             | Conteúdo                                                            |
| ------------------- | ------------------------------------------------------------------- |
| `utils.js`          | Funções auxiliares compartilhadas entre páginas                     |
| `file-validator.js` | Validação de tipo MIME, extensão, tamanho e integridade de arquivos |
| `theme-toggle.js`   | Lógica de alternância entre tema claro e escuro                     |

#### `assets/icons/`

Ícones da aplicação para favicon e instalação PWA.

| Arquivo                 | Uso                           |
| ----------------------- | ----------------------------- |
| `favicon.ico`           | Aba do navegador              |
| `icon-192.png`          | PWA — tela inicial mobile     |
| `icon-512.png`          | PWA — splash screen           |
| `icon-512-maskable.png` | PWA — ícone adaptável Android |

#### `assets/og/`

Imagens Open Graph no formato 1200×630px, uma por ferramenta. Usadas no preview ao compartilhar links no WhatsApp, Facebook, Twitter/X e LinkedIn.

### Arquivos de Configuração

| Arquivo         | Descrição                                               |
| --------------- | ------------------------------------------------------- |
| `manifest.json` | Configuração PWA — nome, ícones, cores, modo standalone |
| `robots.txt`    | Instruções para crawlers — aponta para o sitemap        |
| `sitemap.xml`   | Lista de todas as URLs para indexação no Google         |

---

## `docs/` — Documentação do Projeto

Documentação técnica, decisões de arquitetura, design e planejamento. Faz parte do repositório e é pública — útil para contribuidores e como referência para projetos futuros.

```
docs/
├── sdd/
│   ├── SDD_PDF_Tools_Suite.md
│   └── CHANGELOG.md
├── adr/
│   ├── README.md
│   ├── 001-client-side-only.md
│   ├── 002-catppuccin-palette.md
│   └── 003-no-framework.md
├── design/
│   ├── tokens/
│   │   ├── catppuccin-latte.json
│   │   └── catppuccin-mocha.json
│   ├── wireframes/
│   └── exports/
├── guides/
│   ├── how-to-add-a-tool.md
│   ├── security-checklist.md
│   └── scaling-to-backend.md
└── planning/
    ├── roadmap.md
    └── research-libs.md
```

### `docs/sdd/`

Software Design Document — especificação completa do sistema.

| Arquivo                  | Conteúdo                                                                  |
| ------------------------ | ------------------------------------------------------------------------- |
| `SDD_PDF_Tools_Suite.md` | Documento principal: arquitetura, módulos, segurança, UI/UX, SEO, roadmap |
| `CHANGELOG.md`           | Histórico de versões do próprio SDD — o que mudou em cada revisão         |

### `docs/adr/`

Architecture Decision Records — registros curtos que documentam _por que_ cada decisão de arquitetura foi tomada, não apenas _o que_ foi decidido. Essencial para projetos de estudo: quando revisitar o código meses depois, o raciocínio está preservado.

Formato de cada ADR:

```
# NNN — Título da decisão

**Status:** aceito | deprecado | substituído por ADR-NNN
**Data:** AAAA-MM-DD

## Contexto
O que motivou essa decisão?

## Decisão
O que foi decidido?

## Consequências
O que muda? Quais são os trade-offs?
```

| Arquivo                     | Decisão documentada                       |
| --------------------------- | ----------------------------------------- |
| `README.md`                 | O que é ADR e como criar novos registros  |
| `001-client-side-only.md`   | Processar tudo no navegador, sem servidor |
| `002-catppuccin-palette.md` | Adotar Catppuccin como sistema de cores   |
| `003-no-framework.md`       | Usar JavaScript vanilla sem framework SPA |

### `docs/design/`

Artefatos visuais do sistema de design.

| Pasta / Arquivo                | Conteúdo                                              |
| ------------------------------ | ----------------------------------------------------- |
| `tokens/catppuccin-latte.json` | Tokens de cor do tema claro em formato JSON           |
| `tokens/catppuccin-mocha.json` | Tokens de cor do tema escuro em formato JSON          |
| `wireframes/`                  | Arquivos de design (.fig, .sketch ou .xd)             |
| `exports/`                     | PNGs exportados dos wireframes para referência rápida |

### `docs/guides/`

Guias práticos e reutilizáveis. A ideia é que cada guia seja genérico o suficiente para ser aproveitado em projetos futuros.

| Arquivo                 | Conteúdo                                                                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `how-to-add-a-tool.md`  | Passo a passo para criar uma nova ferramenta: estrutura HTML, CSS necessário, bibliotecas, checklist de qualidade                                 |
| `security-checklist.md` | Lista de verificação de segurança: CSP, validação de entrada, privacidade, SRI em CDNs                                                            |
| `scaling-to-backend.md` | O que mudaria para evoluir para uma arquitetura com servidor e banco de dados: autenticação, upload de arquivos, histórico, fila de processamento |

### `docs/planning/`

Registros de planejamento e pesquisa.

| Arquivo            | Conteúdo                                                                                       |
| ------------------ | ---------------------------------------------------------------------------------------------- |
| `roadmap.md`       | Fases de desenvolvimento, critérios de aceite por ferramenta, status atual                     |
| `research-libs.md` | Notas de avaliação das bibliotecas consideradas — por que cada uma foi escolhida ou descartada |

---

## Convenções

### Nomenclatura de arquivos

- Páginas HTML: `ação-formato.html` ou `formato-para-formato.html`
- Documentação: `kebab-case.md` para guias, `NNN-titulo.md` para ADRs
- Assets: `kebab-case` em todas as pastas

### Commits

Seguir o padrão [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: adiciona ferramenta ocr-pdf
fix: corrige validação de MIME em file-validator.js
docs: atualiza SDD com seção de riscos
style: aplica tokens Catppuccin em components.css
chore: adiciona .editorconfig
```

### Branches

```
main          → produção estável
dev           → integração de features
feat/nome     → nova ferramenta ou funcionalidade
fix/nome      → correção de bug
docs/nome     → atualização de documentação
```

---

_PDF Tools Suite — PROJECT_STRUCTURE.md_
