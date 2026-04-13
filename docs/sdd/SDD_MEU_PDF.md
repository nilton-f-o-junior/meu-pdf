# Meu PDF

**Versão:** 1.0 (Draft Inicial)
**Status:** Em desenvolvimento
**Data:** Abril 2025
**Tipo:** Aplicação Web Client-Side

---

## Sumário

1. [Introdução e Visão Geral](#1-introdução-e-visão-geral)
2. [Arquitetura do Sistema](#2-arquitetura-do-sistema)
3. [Especificação dos Módulos](#3-especificação-dos-módulos)
4. [Segurança](#4-segurança)
5. [Design de Interface (UI/UX)](#5-design-de-interface-uiux)
6. [SEO, PWA e Performance](#6-seo-pwa-e-performance)
7. [Roadmap de Implementação](#7-roadmap-de-implementação)
8. [Riscos e Mitigações](#8-riscos-e-mitigações)
9. [Referências Técnicas](#9-referências-técnicas)

---

## 1. Introdução e Visão Geral

### 1.1 Objetivo do Documento

Este SDD (Software Design Document) descreve a arquitetura, os componentes, os módulos e as decisões de design do **PDF Tools Suite** — uma suíte web de ferramentas para manipulação de documentos PDF e conversão de formatos Office, executada inteiramente no navegador do usuário (client-side), sem envio de dados para servidores externos.

### 1.2 Escopo do Sistema

O PDF Tools Suite é uma coleção de páginas HTML independentes, cada uma responsável por uma função específica de manipulação de documentos. O sistema faz processamento local via JavaScript, garantindo privacidade total dos arquivos do usuário.

> **Princípio Fundamental de Design**
> Todo o processamento ocorre no navegador (client-side). Nenhum arquivo é transmitido a servidores. Cada funcionalidade é uma página HTML autônoma e independente. A aplicação não requer backend, banco de dados ou autenticação.

### 1.3 Público-Alvo

Usuários finais que necessitam manipular PDFs e documentos Office sem instalar software local e sem comprometer a privacidade de seus arquivos — profissionais, estudantes e usuários domésticos.

### 1.4 Definições e Acrônimos

| Termo     | Definição                                                               |
| --------- | ----------------------------------------------------------------------- |
| SPA       | Single Page Application — aplicação de página única                     |
| CSP       | Content Security Policy — política de segurança de conteúdo HTTP        |
| PWA       | Progressive Web App — app web instalável no dispositivo                 |
| OCR       | Optical Character Recognition — reconhecimento ótico de caracteres      |
| JSON-LD   | JSON for Linked Data — formato de dados estruturados para SEO           |
| DXA / EMU | Unidades de medida internas de documentos Office                        |
| Blob URL  | URL temporária criada no navegador para download de arquivo gerado      |
| FOUT      | Flash of Unstyled Text — carregamento visível de fontes não estilizadas |

---

## 2. Arquitetura do Sistema

### 2.1 Visão Arquitetural

A arquitetura segue o padrão de **Micro-Frontends independentes**. Não há roteamento central, estado compartilhado ou framework SPA. Cada ferramenta é um arquivo HTML completo e autossuficiente, reutilizando apenas assets estáticos (CSS global, fontes, ícones).

**Camadas:**

- **Apresentação:** HTML + CSS (tema claro/escuro, responsividade, menu de navegação)
- **Lógica:** JavaScript vanilla + bibliotecas especializadas (sem framework)
- **I/O:** File API do navegador (leitura), Blob URL (geração de download)
- **Segurança:** CSP headers, processamento local, sem requisições externas de dados

### 2.2 Estrutura de Arquivos

```
/
├── index.html                  # Home — apresentação, privacy banner, menu de ferramentas
├── juntar-pdf.html             # Mesclar múltiplos PDFs em um único arquivo
├── dividir-pdf.html            # Extrair páginas selecionadas de um PDF
├── comprimir-pdf.html          # Reduzir tamanho de arquivo PDF
├── pdf-para-word.html          # Converter PDF em documento .docx
├── word-pdf.html               # Converter .docx/.doc em PDF
├── pdf-para-jpg.html           # Extrair páginas como imagens JPG/PNG
├── jpg-pdf.html                # Converter imagens em PDF
├── pdf-para-pptx.html          # Converter PDF em apresentação PowerPoint
├── pptx-pdf.html               # Converter .pptx em PDF
├── pdf-para-excel.html         # Extrair tabelas de PDF para .xlsx
├── excel-pdf.html              # Converter planilha .xlsx em PDF
├── ocr-pdf.html                # Reconhecimento de texto em PDFs digitalizados
├── assinar-pdf.html            # Adicionar assinatura digital em PDF
├── marca-dagua.html            # Adicionar marca d'água em PDFs
├── numerar-paginas.html        # Adicionar numeração de páginas
├── manifest.json               # Configuração do Progressive Web App
└── assets/
    ├── css/
    │   ├── global.css          # Reset, variáveis CSS, tipografia, tema claro/escuro
    │   └── components.css      # Componentes reutilizáveis: dropzone, botões, progress
    └── js/
        └── utils.js            # Funções utilitárias compartilhadas
```

### 2.3 Dependências por Funcionalidade

Cada página carrega somente as bibliotecas necessárias para sua função, evitando o carregamento desnecessário de código.

| Biblioteca          | Versão Recomendada | Funcionalidades                                            |
| ------------------- | ------------------ | ---------------------------------------------------------- |
| pdf-lib             | ^1.17              | Juntar, dividir, comprimir, marca d'água, numerar, assinar |
| PDF.js (pdfjs-dist) | ^4.x               | Visualização, extração de texto, converter PDF→JPG         |
| jsPDF               | ^2.5               | Geração de PDF a partir de imagens/HTML                    |
| JSZip               | ^3.10              | Manipulação de .docx, .pptx, .xlsx (são ZIPs internamente) |
| docx.js             | ^8.x               | Geração de documentos Word (.docx)                         |
| xlsx (SheetJS)      | ^0.18              | Leitura e escrita de planilhas Excel (.xlsx)               |
| Tesseract.js        | ^5.x               | OCR — reconhecimento de texto em imagens/PDF               |
| mammoth.js          | ^1.6               | Conversão Word → HTML para extração de conteúdo            |

---

## 3. Especificação dos Módulos

### 3.1 `juntar-pdf.html`

Permite ao usuário selecionar dois ou mais arquivos PDF e combiná-los em um único documento, respeitando a ordem definida pelo usuário via drag-and-drop na lista.

**Fluxo de funcionamento:**

1. Usuário arrasta arquivos ou usa o seletor de arquivos (File API)
2. Lista de arquivos renderizada com pré-visualização da primeira página (PDF.js)
3. Usuário reordena arquivos por drag-and-drop
4. Ao clicar em "Juntar", pdf-lib carrega todos os PDFs como `ArrayBuffer`
5. Páginas são copiadas na ordem definida para um novo `PDFDocument`
6. Documento final é serializado e oferecido como download via Blob URL

**Estados da interface:**

| Estado      | Comportamento                                   |
| ----------- | ----------------------------------------------- |
| Vazio       | Dropzone com instruções visíveis                |
| Carregando  | Barra de progresso por arquivo                  |
| Pronto      | Lista reordenável, botão de ação habilitado     |
| Processando | Progress bar global, botão desabilitado         |
| Concluído   | Botão de download + opção de novo processamento |

---

### 3.2 `word-pdf.html`

Converte documentos `.docx` e `.doc` em PDF diretamente no navegador. Dado que a conversão fiel de Word para PDF é complexa (fontes embutidas, estilos avançados), esta ferramenta utiliza uma abordagem híbrida de renderização HTML intermediária.

**Estratégia técnica:**

1. `mammoth.js` converte o `.docx` em HTML semântico (preservando estrutura de títulos, listas, tabelas)
2. O HTML é renderizado em um `<iframe>` oculto com estilos de impressão otimizados
3. `jsPDF` com plugin `html()` captura o conteúdo e gera o PDF paginado

> **Limitação conhecida:** fontes customizadas embutidas no `.docx` podem não ser preservadas na conversão.

---

### 3.3 `ocr-pdf.html`

Executa reconhecimento ótico de caracteres em PDFs digitalizados (sem texto selecionável), utilizando Tesseract.js com suporte a múltiplos idiomas, incluindo Português.

**Fluxo de funcionamento:**

1. PDF carregado e páginas rasterizadas em `<canvas>` via PDF.js
2. Cada canvas exportado como `ImageData` e enviado ao Tesseract.js Worker
3. Workers rodam em Web Workers (thread separada, não bloqueia a UI)
4. Texto reconhecido é adicionado como camada invisível sobre cada página
5. pdf-lib gera o PDF final com texto pesquisável

**Configurações de performance:**

- Tesseract Worker é inicializado uma única vez por sessão
- Processamento de páginas em paralelo (máximo 4 workers simultâneos)
- Progress report por página com callback para atualizar a UI

---

### 3.4 Módulos de Conversão Office → PDF

Todos os formatos Office modernos (`.xlsx`, `.pptx`, `.docx`) são internamente arquivos ZIP contendo XML. A estratégia de conversão segue um padrão consistente:

| Ferramenta       | Biblioteca de Leitura | Estratégia de PDF                                     |
| ---------------- | --------------------- | ----------------------------------------------------- |
| `excel-pdf.html` | SheetJS (xlsx)        | Renderiza tabela em canvas → jsPDF                    |
| `pptx-pdf.html`  | JSZip + XML parsing   | Renderiza cada slide em canvas → jsPDF                |
| `word-pdf.html`  | mammoth.js            | HTML intermediário → jsPDF `html()`                   |
| `jpg-pdf.html`   | FileReader API        | Imagens → jsPDF `addImage()` com paginação automática |

---

## 4. Segurança

### 4.1 Content Security Policy (CSP)

Cada página deve incluir o seguinte cabeçalho CSP via meta tag, prevenindo injeção de scripts maliciosos:

```
default-src 'self';
script-src 'self' cdn.jsdelivr.net unpkg.com cdnjs.cloudflare.com 'wasm-unsafe-eval';
style-src 'self' 'unsafe-inline' fonts.googleapis.com;
font-src 'self' fonts.gstatic.com;
worker-src 'self' blob:;
img-src 'self' blob: data:;
connect-src 'self';
```

> `'wasm-unsafe-eval'` é necessário para Tesseract.js e PDF.js (módulos WebAssembly).

### 4.2 Privacidade e Proteção de Dados

- Nenhum arquivo é enviado a servidores externos — todo processamento ocorre no navegador
- Blob URLs são revogadas imediatamente após o download (`URL.revokeObjectURL`)
- Nenhum dado é persistido em `localStorage` ou `IndexedDB` sem consentimento explícito
- O Privacy Banner na homepage comunica claramente a política de dados
- Sem cookies de rastreamento ou analytics de terceiros

### 4.3 Validação de Entrada

- Validação de tipo MIME e extensão antes do processamento
- Limite de tamanho por arquivo: **50 MB** (configurável por ferramenta)
- Validação de integridade do PDF antes de operações com pdf-lib
- Tratamento de exceções para arquivos corrompidos com mensagens amigáveis

---

## 5. Design de Interface (UI/UX)

### 5.1 Sistema de Design

A interface utiliza um sistema de design próprio baseado em **CSS Custom Properties**, sem dependência de frameworks externos. A paleta adotada é o **Catppuccin** — nos flavours **Latte** (tema claro) e **Mocha** (tema escuro).

> Catppuccin é uma paleta pastel de baixo contraste, organizada em quatro flavours do mais claro ao mais escuro: Latte → Frappé → Macchiato → Mocha. O projeto é open-source: https://github.com/catppuccin/catppuccin

#### Cores de Superfície

| Token CSS        | Latte (claro) | Mocha (escuro) | Papel                            |
| ---------------- | ------------- | -------------- | -------------------------------- |
| `--ctp-base`     | `#EFF1F5`     | `#1E1E2E`      | Fundo da página                  |
| `--ctp-mantle`   | `#E6E9EF`     | `#181825`      | Fundo alternativo / sidebar      |
| `--ctp-crust`    | `#DCE0E8`     | `#11111B`      | Fundo mais profundo / footer     |
| `--ctp-surface0` | `#CCD0DA`     | `#313244`      | Cards, painéis, inputs           |
| `--ctp-surface1` | `#BCC0CC`     | `#45475A`      | Bordas, separadores              |
| `--ctp-surface2` | `#ACB0BE`     | `#585B70`      | Hover de superfícies             |
| `--ctp-overlay0` | `#9CA0B0`     | `#6C7086`      | Texto desabilitado / placeholder |
| `--ctp-overlay1` | `#8C8FA1`     | `#7F849C`      | Texto secundário                 |
| `--ctp-overlay2` | `#7C7F93`     | `#9399B2`      | Texto terciário / ícones         |

#### Cores de Texto

| Token CSS        | Latte (claro) | Mocha (escuro) | Papel            |
| ---------------- | ------------- | -------------- | ---------------- |
| `--ctp-text`     | `#4C4F69`     | `#CDD6F4`      | Texto principal  |
| `--ctp-subtext1` | `#5C5F77`     | `#BAC2DE`      | Texto secundário |
| `--ctp-subtext0` | `#6C6F85`     | `#A6ADC8`      | Labels, captions |

#### Cores de Acento

| Token CSS         | Latte (claro) | Mocha (escuro) | Uso na UI                            |
| ----------------- | ------------- | -------------- | ------------------------------------ |
| `--ctp-blue`      | `#1E66F5`     | `#89B4FA`      | Botões primários, links, foco        |
| `--ctp-lavender`  | `#7287FD`     | `#B4BEFE`      | Destaques suaves, badges info        |
| `--ctp-mauve`     | `#8839EF`     | `#CBA6F7`      | Acento secundário, tags              |
| `--ctp-green`     | `#40A02B`     | `#A6E3A1`      | Sucesso, upload concluído            |
| `--ctp-teal`      | `#179299`     | `#94E2D5`      | Progresso, estados intermediários    |
| `--ctp-sky`       | `#04A5E5`     | `#89DCEB`      | Links secundários, ícones            |
| `--ctp-yellow`    | `#DF8E1D`     | `#F9E2AF`      | Alertas, avisos                      |
| `--ctp-peach`     | `#FE640B`     | `#FAB387`      | Avisos de atenção                    |
| `--ctp-red`       | `#D20F39`     | `#F38BA8`      | Erros, validações negativas          |
| `--ctp-maroon`    | `#E64553`     | `#EBA0AC`      | Erros secundários                    |
| `--ctp-flamingo`  | `#DD7878`     | `#F2CDCD`      | Elementos decorativos suaves         |
| `--ctp-rosewater` | `#DC8A78`     | `#F5E0DC`      | Detalhes ornamentais                 |
| `--ctp-sapphire`  | `#209FB5`     | `#74C7EC`      | Links visitados, seleções            |
| `--ctp-pink`      | `#EA76CB`     | `#F5C2E7`      | Elementos especiais, destaque lúdico |

#### Mapeamento Semântico

| Papel na UI                  | Token            | Justificativa                                       |
| ---------------------------- | ---------------- | --------------------------------------------------- |
| Ação primária (botão, CTA)   | `--ctp-blue`     | Cor mais neutra e universalmente associada a "ação" |
| Sucesso / arquivo processado | `--ctp-green`    | Verde semântico convencional                        |
| Aviso / arquivo grande       | `--ctp-yellow`   | Amarelo de atenção                                  |
| Erro / arquivo inválido      | `--ctp-red`      | Vermelho semântico convencional                     |
| Progresso / upload           | `--ctp-teal`     | Tom frio e calmo para operações em andamento        |
| Acento / destaque de marca   | `--ctp-mauve`    | Cor característica do Catppuccin                    |
| Fundo de cards               | `--ctp-surface0` | Separação sutil em relação ao fundo da página       |
| Bordas                       | `--ctp-surface1` | Um tom acima do card para contraste mínimo          |

### 5.2 Componentes Reutilizáveis

**Drop Zone**
Componente de arrastar e soltar arquivos, presente em todas as ferramentas. Aceita clique para abrir seletor de arquivos. Exibe feedback visual ao arrastar (border highlight + overlay).

**Progress Bar**
Barra de progresso com porcentagem numérica. Suporta modo indeterminado (animação de pulso) para operações sem progresso mensurável, como inicialização do Tesseract.js.

**File Preview Card**
Card com miniatura da primeira página do PDF, nome do arquivo, tamanho em KB/MB e botão de remoção. Suporta drag-and-drop para reordenação na lista.

**Toast Notification**
Notificações temporárias (3–5 segundos) para feedback de ações — sucesso, erro ou informação. Posicionadas no canto inferior direito, empilháveis.

### 5.3 Responsividade

| Breakpoint | Largura        | Comportamento                                                    |
| ---------- | -------------- | ---------------------------------------------------------------- |
| Mobile     | < 768px        | Menu hamburguer, layout em coluna única, dropzone compacta       |
| Tablet     | 768px – 1024px | Menu horizontal reduzido, grid 2 colunas para ferramentas        |
| Desktop    | > 1024px       | Menu completo com dropdowns, grid 3–4 colunas, sidebar de opções |

### 5.4 Acessibilidade (a11y)

- Todos os elementos interativos possuem `aria-label` descritivo
- Navegação completa por teclado (Tab, Enter, Espaço, setas)
- Contraste mínimo WCAG AA (4.5:1) garantido nos dois temas
- Mensagens de status anunciadas para leitores de tela via `aria-live`
- Imagens decorativas com `alt=""` e imagens informativas com alt descritivo

---

## 6. SEO, PWA e Performance

### 6.1 Estrutura de Metadados

Cada página HTML deve incluir o conjunto completo de metadados:

| Meta Tag                      | Descrição                                                         |
| ----------------------------- | ----------------------------------------------------------------- |
| `<title>`                     | Formato: `[Ação] PDF Online Grátis \| PDF Tools Suite`            |
| `<meta description>`          | 150–160 caracteres descrevendo a ferramenta e benefício principal |
| `og:title` / `og:description` | Versão otimizada para compartilhamento no WhatsApp/Facebook       |
| `og:image`                    | Imagem de 1200×630px específica por ferramenta                    |
| `twitter:card`                | `summary_large_image` para preview expandido no Twitter/X         |
| `<link rel="canonical">`      | URL canônica para evitar conteúdo duplicado                       |
| JSON-LD `WebApplication`      | Schema markup para o Google Knowledge Panel de apps               |
| `<link rel="manifest">`       | Referência ao `manifest.json` para instalação PWA                 |

### 6.2 Progressive Web App (PWA)

Configuração do `manifest.json`:

```json
{
  "name": "PDF Tools Suite",
  "short_name": "PDF Tools",
  "start_url": "/index.html",
  "display": "standalone",
  "theme_color": "#2563EB",
  "background_color": "#FFFFFF",
  "icons": [
    { "src": "icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    {
      "src": "icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ]
}
```

### 6.3 Performance

**Carregamento de fontes:**

- `preconnect` para `fonts.googleapis.com` e `fonts.gstatic.com` no `<head>`
- `font-display: swap` para evitar FOUT e não bloquear renderização
- `preload` da fonte mais crítica com `<link rel="preload">`

**Carregamento de bibliotecas:**

- Carregar bibliotecas apenas na página que as utiliza (sem bundle global)
- Usar `defer` ou `async` em scripts não críticos para o render inicial
- CDN com SRI (Subresource Integrity) hash para segurança e cache eficiente
- Web Workers para processamento pesado (Tesseract, PDF rendering) — não bloqueiam a UI

---

## 7. Roadmap de Implementação

### 7.1 Fases de Desenvolvimento

| Fase                        | Entregas                                                                     | Prioridade |
| --------------------------- | ---------------------------------------------------------------------------- | ---------- |
| Fase 1 — Núcleo             | `index.html` + `juntar-pdf.html` + `dividir-pdf.html` + `comprimir-pdf.html` | Alta       |
| Fase 2 — Conversões Básicas | `word-pdf.html` + `jpg-pdf.html` + `pdf-para-jpg.html`                       | Alta       |
| Fase 3 — Office Suite       | `excel-pdf.html` + `pptx-pdf.html` + `pdf-para-excel.html`                   | Média      |
| Fase 4 — Avançado           | `ocr-pdf.html` + `assinar-pdf.html` + `marca-dagua.html`                     | Média      |
| Fase 5 — Polimento          | PWA completo, testes a11y, otimização de performance, SEO final              | Baixa      |

### 7.2 Critérios de Aceite por Ferramenta

Cada ferramenta deve atender aos seguintes critérios antes de ser considerada pronta:

- Funciona em Chrome, Firefox, Safari e Edge (versões dos últimos 2 anos)
- Funciona em dispositivos móveis iOS e Android
- Processa arquivo de 10 MB em menos de 10 segundos em hardware médio
- Exibe mensagem de erro clara para arquivos inválidos ou corrompidos
- Score Lighthouse > 90 em Performance, Acessibilidade e SEO
- Nenhuma requisição de rede é feita com o conteúdo do arquivo do usuário

---

## 8. Riscos e Mitigações

| Risco                                                       | Probabilidade | Impacto | Mitigação                                                                      |
| ----------------------------------------------------------- | ------------- | ------- | ------------------------------------------------------------------------------ |
| Limite de memória do navegador para PDFs grandes (> 100 MB) | Média         | Alto    | Chunked processing + aviso de limite ao usuário                                |
| Conversão Word→PDF infiel (fontes, layout complexo)         | Alta          | Médio   | Documentar limitações claramente na UI; oferecer resultado "best effort"       |
| Tesseract.js lento em dispositivos de baixo desempenho      | Média         | Médio   | Web Workers + progress feedback + opção de qualidade reduzida                  |
| CSP quebrar bibliotecas com `eval()`                        | Média         | Alto    | `'wasm-unsafe-eval'` como fallback controlado; testar cada lib individualmente |
| Safari com suporte limitado a APIs de File/Worker           | Baixa         | Médio   | Testes específicos + polyfills seletivos para APIs críticas                    |

---

## 9. Referências Técnicas

- **pdf-lib:** https://pdf-lib.js.org/
- **PDF.js:** https://mozilla.github.io/pdf.js/
- **Tesseract.js:** https://tesseract.projectnaptha.com/
- **mammoth.js:** https://github.com/mwilliamson/mammoth.js
- **SheetJS:** https://sheetjs.com/
- **jsPDF:** https://artskydj.github.io/jsPDF/
- **Content Security Policy (MDN):** https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- **Web App Manifest (MDN):** https://developer.mozilla.org/en-US/docs/Web/Manifest
- **WCAG 2.1 Guidelines:** https://www.w3.org/TR/WCAG21/
- **Schema.org WebApplication:** https://schema.org/WebApplication

---

_PDF Tools Suite — SDD v1.0_
