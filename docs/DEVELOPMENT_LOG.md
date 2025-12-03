# Development Log - Product Images Custom

Este documento registra as customizações e melhorias implementadas no app `product-images-custom` durante o desenvolvimento.

---

## 📅 Data: [Data de hoje]

### 🎯 Objetivos da Sessão
- Customização do comportamento do carrossel de thumbnails
- Implementação de CSS condicional baseado em props
- Melhorias na experiência do usuário com loop infinito
- Ajustes de responsividade e tamanhos
- Correção de bugs de sincronização e comportamento visual

---

## ✅ Mudanças Implementadas

### 1. Localização dos Arquivos CSS Globais

**Arquivos identificados:**
- `react/components/ProductImagesCustom/components/Gallery/global.css` - Estilos do PhotoSwipe
- `react/components/ProductImagesCustom/components/Carousel/swiper.global.css` - Estilos base do Swiper
- `react/components/ProductImagesCustom/components/Carousel/overrides.global.css` - Overrides customizados
- `react/components/ProductImagesCustom/styles.css` - CSS Module (não global)

**Localização das importações:**
- `Gallery/global.css` importado em `Gallery/Gallery.js` (linha 4)
- `swiper.global.css` e `overrides.global.css` importados em `Carousel/index.js` (linhas 22-23)

---

### 2. CSS Condicional Baseado em `.hideFirstImage`

**Problema:** Aplicar CSS apenas quando a classe `.hideFirstImage` não existe no documento.

**Solução implementada (Store Theme):**
/* Aplicar quando não existe nenhum .hideFirstImage no documento */
body:not(:has(.hideFirstImage)) * {
  --header-text-color: #161413;
}**Arquivo:** Store Theme CSS (não no app)

**Comportamento:**
- Quando `hideFirstImage` prop é `false` (botão desativado): CSS é aplicado
- Quando `hideFirstImage` prop é `true` (botão ativado): CSS não é aplicado

**Nota:** Solução implementada diretamente no store-theme usando a pseudo-classe `:has()` do CSS moderno.

---

### 3. Customização do Carrossel de Thumbnails - 3 Espaços Visuais Sempre

**Problema:** Garantir que sempre sejam exibidos 3 espaços visuais (ocupados ou vazios), cada um ocupando exatamente 1/3 do espaço disponível.

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Mudanças:**
- Linha 178: `slidesPerView={3}` - Sempre mostra 3 espaços visuais
- Removida lógica condicional de `slidesPerView`
- Removida lógica de largura fixa (`thumbWidth`) quando há menos de 3 slides
- Linha 193: `centeredSlides={slides.length < 3}` - Centraliza quando há menos de 3 slides
- Linha 194: `centeredSlidesBounds={slides.length < 3}` - Limita bounds quando centralizado

**Comportamento:**
- **Sempre:** Mostra 3 espaços visuais, cada um ocupando 1/3 do espaço
- **1 slide:** 1 thumbnail ocupando espaço central, 2 espaços vazios
- **2 slides:** 2 thumbnails centralizados, 1 espaço vazio
- **3+ slides:** 3 thumbnails visíveis, scroll horizontal para ver mais

**CSS complementar (Store Theme ou swiper.scoped.css):**
/* Garantir que cada slide ocupe exatamente 1/3 do espaço */
.carouselGaleryThumbs .swiper-slide {
  width: calc((100% - 20px) / 3) !important; /* 100% - (2 * spaceBetween) / 3 */
  flex-shrink: 0;
  flex-grow: 0;
}---

### 4. Loop Infinito nos Carrosséis

**Arquivo modificado:** 
- `react/components/ProductImagesCustom/components/Carousel/index.js` (Swiper principal)
- `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js` (Thumbnails)

**Mudanças no Swiper Principal:**
- Linha 352-353: `loop={slides.length > 1}` e `loopedSlides={slides.length >= 3 ? 3 : slides.length}`
- **Comportamento:** Loop sempre ativo quando há mais de 1 slide

**Mudanças no ThumbnailSwiper:**
- Linha 189: `loop={false}` - **Loop desabilitado permanentemente**
- **Motivo:** Evitar problemas de sincronização entre carrossel principal e thumbnails

**Resultado:** Carrossel principal sempre infinito (quando aplicável), thumbnails sem loop para evitar bugs de sincronização.

---

### 5. Renderização de Thumbnails com 1 Slide

**Problema:** Thumbnails não eram renderizados quando havia apenas 1 imagem.

**Arquivos modificados:**
- `react/components/ProductImagesCustom/components/Carousel/index.js`
- `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Mudanças:**
- `index.js` linha 266: `hasThumbs = slides && slides.length >= 1` (antes: `> 1`)
- `ThumbnailSwiper.js` linha 73: `hasThumbs = slides.length >= 1` (antes: `> 1`)
- Adicionadas verificações `hasThumbs &&` nas condições de renderização

**Resultado:** Thumbnails são renderizados mesmo quando há apenas 1 slide.

---

### 6. Desabilitar Navegação dos Thumbnails com Menos de 3 Slides

**Problema:** Setas de navegação dos thumbnails causavam bugs quando havia menos de 3 slides.

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Mudanças:**
- Linha 77: `const shouldShowNavigation = slidesCount >= 3 && displayThumbnailsArrows`
- Linha 185: `navigation={shouldShowNavigation ? navigationConfig : false}`
- Linha 116: Adicionada verificação `slidesCount < 3` no `useMemo` das arrows
- Linha 169: Adicionado `slidesCount` como dependência do `useMemo`

**Comportamento:**
- **3+ slides:** Navegação habilitada (se `displayThumbnailsArrows` for `true`)
- **1-2 slides:** Navegação desabilitada

**Resultado:** Elimina bugs de sincronização quando há poucos slides.

---

### 7. Desabilitar Drag no Desktop com 2 ou Menos Slides

**Problema:** Drag (arrastar com mouse) no desktop causava comportamento estranho quando havia 2 slides, jogando os slides para o final do carrossel.

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Mudanças:**
- Linha 80: `const isDesktop = typeof window !== 'undefined' && window.innerWidth >= 640`
- Linha 82: `const shouldDisableDrag = isDesktop && slidesCount <= 2`
- Linha 191: `simulateTouch={!shouldDisableDrag}` - Desabilita simulação de touch no desktop
- Linha 192: `allowTouchMove={!shouldDisableDrag}` - Desabilita movimento via touch

**Comportamento:**
- **Desktop com 1-2 slides:** Drag desabilitado
- **Desktop com 3+ slides:** Drag habilitado
- **Mobile (qualquer quantidade):** Drag habilitado (necessário para navegação)

**Resultado:** Elimina comportamento estranho do drag no desktop quando há poucos slides, mantendo funcionalidade no mobile.

---

## 📝 Notas Técnicas

### Arquivos CSS Globais
- Arquivos com sufixo `.global.css` são aplicados globalmente quando importados
- Arquivos `.scoped.css` são CSS Modules (classes scoped)
- Arquivo `styles.css` é usado como CSS Module em vários componentes

### Swiper Configuration
- `slidesPerView`: Número de slides visíveis simultaneamente
- `slidesPerGroup`: Número de slides que avançam por transição
- `loop`: Habilita loop infinito (desabilitado nas thumbnails para evitar bugs)
- `loopedSlides`: Número de slides duplicados para o loop funcionar
- `spaceBetween`: Espaçamento entre slides (em pixels) - sempre 10px
- `centeredSlides`: Centraliza slides quando há menos que o `slidesPerView`
- `centeredSlidesBounds`: Limita os bounds quando centralizado
- `simulateTouch`: Simula eventos de touch no desktop (mouse drag)
- `allowTouchMove`: Permite movimento via touch/drag

### Breakpoints
- Desktop: `>= 640px` (40em)
- Mobile: `< 640px`

### Fórmula de Largura dos Thumbnails
Com `slidesPerView={3}` e `spaceBetween={10}`:
- Cada slide ocupa: `calc((100% - 20px) / 3)`
- Onde `20px = 2 * spaceBetween` (espaço entre 3 slides = 2 espaços)

---

## 🐛 Bugs Corrigidos

### Bug 1: Sincronização Reversa entre Carrosséis
**Problema:** Ao navegar no carrossel principal, o carrossel de thumbnails acompanhava na direção errada.

**Solução:** Desabilitar loop infinito nas thumbnails (`loop={false}`)

**Status:** ✅ Resolvido

### Bug 2: Comportamento Estranho com 2 Slides
**Problema:** Com 2 slides, drag no desktop causava comportamento estranho, jogando slides para o final.

**Solução:** Desabilitar `simulateTouch` e `allowTouchMove` no desktop quando há 2 ou menos slides.

**Status:** ✅ Resolvido

### Bug 3: Navegação dos Thumbnails com Poucos Slides
**Problema:** Setas de navegação dos thumbnails causavam bugs quando havia menos de 3 slides.

**Solução:** Desabilitar navegação quando `slidesCount < 3`.

**Status:** ✅ Resolvido

---

## 🔄 Próximos Passos

- [x] Ajustar comportamento de sincronização entre carrossel principal e thumbnails
- [ ] Testar em diferentes dispositivos e navegadores
- [ ] Validar performance com muitos slides
- [ ] Documentar props adicionais no README
- [ ] Adicionar CSS para garantir 1/3 do espaço sempre (se necessário)

---

## 📚 Referências

- [Swiper.js Documentation](https://swiperjs.com/)
- [CSS :has() Selector](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)
- VTEX IO CSS Handles Documentation
