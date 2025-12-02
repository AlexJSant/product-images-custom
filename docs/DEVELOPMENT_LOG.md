# Development Log - Product Images Custom

Este documento registra as customizações e melhorias implementadas no app `product-images-custom` durante o desenvolvimento.

---

## 📅 Data: [Data de hoje]

### 🎯 Objetivos da Sessão
- Customização do comportamento do carrossel de thumbnails
- Implementação de CSS condicional baseado em props
- Melhorias na experiência do usuário com loop infinito
- Ajustes de responsividade e tamanhos

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

### 3. Customização do Carrossel de Thumbnails - 3 Slides Fixos

**Problema:** Mostrar 3 thumbnails fixos no render inicial quando há 3 ou mais slides.

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Mudanças:**
- Linha 168: `slidesPerView={slides.length >= 3 ? 3 : "auto"}`
  - Com 3+ slides: mostra exatamente 3 thumbnails
  - Com menos de 3: usa `"auto"` para respeitar largura CSS

**Comportamento:**
- **3+ slides:** Mostra 3 thumbnails fixos no render inicial
- **1-2 slides:** Mostra todos os thumbnails disponíveis

---

### 4. Loop Infinito nos Carrosséis

**Arquivo modificado:** 
- `react/components/ProductImagesCustom/components/Carousel/index.js` (Swiper principal)
- `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js` (Thumbnails)

**Mudanças no Swiper Principal:**
- Linha 352-353: `loop={slides.length > 1}` e `loopedSlides={slides.length >= 3 ? 3 : slides.length}`
- **Comportamento:** Loop sempre ativo quando há mais de 1 slide

**Mudanças no ThumbnailSwiper:**
- Linha 174-175: `loop={slides.length >= 3}` e `loopedSlides={slides.length >= 3 ? 3 : undefined}`
- **Comportamento:** 
  - **3+ slides:** Loop ativo
  - **1-2 slides:** Loop desativado

**Resultado:** Carrossel principal sempre infinito (quando aplicável), thumbnails com loop apenas quando há 3+ slides.

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

### 6. Largura Mínima dos Thumbnails

**Problema:** Quando há menos de 3 slides, os thumbnails ficavam muito pequenos usando apenas `w-20` (20%).

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Solução implementada:**ipt
// Definir largura mínima dos thumbnails
const THUMB_MIN_WIDTH_DESKTOP = 231px
const THUMB_MIN_WIDTH_MOBILE = 100px

// Calcular largura baseada no número de slides
const getThumbWidth = () => {
  if (slidesCount >= 3) {
    return undefined // Usa w-20 (20%) quando há 3+ slides
  }
  // Quando há menos de 3, usa largura mínima
  return typeof window !== 'undefined' && window.innerWidth >= 640 
    ? `${THUMB_MIN_WIDTH_DESKTOP}px` 
    : `${THUMB_MIN_WIDTH_MOBILE}px`
}**Aplicado no style do SwiperSlide:**
- `width: thumbWidth`
- `minWidth: thumbWidth`

**Comportamento:**
- **3+ slides:** Usa `w-20` (20% da largura) - 3 thumbnails por vez
- **1-2 slides:** Usa largura fixa (231px desktop / 100px mobile)

---

## 📝 Notas Técnicas

### Arquivos CSS Globais
- Arquivos com sufixo `.global.css` são aplicados globalmente quando importados
- Arquivos `.scoped.css` são CSS Modules (classes scoped)
- Arquivo `styles.css` é usado como CSS Module em vários componentes

### Swiper Configuration
- `slidesPerView`: Número de slides visíveis simultaneamente
- `slidesPerGroup`: Número de slides que avançam por transição
- `loop`: Habilita loop infinito
- `loopedSlides`: Número de slides duplicados para o loop funcionar
- `spaceBetween`: Espaçamento entre slides (em pixels)

### Breakpoints
- Desktop: `>= 640px` (40em)
- Mobile: `< 640px`

---

## 🔄 Próximos Passos

- [ ] Ajustar comportamento de sincronização entre carrossel principal e thumbnails
- [ ] Testar em diferentes dispositivos e navegadores
- [ ] Validar performance com muitos slides
- [ ] Documentar props adicionais no README

---

## 📚 Referências

- [Swiper.js Documentation](https://swiperjs.com/)
- [CSS :has() Selector](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)
- VTEX IO CSS Handles Documentation
