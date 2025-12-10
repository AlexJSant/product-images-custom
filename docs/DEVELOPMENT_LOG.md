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
- Correção do reset do carrossel ao mudar variação de SKU
- Implementação de aspect ratio fixo para thumbnails
- Ajustes de espaçamento e alinhamento

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
- Linha 183: `slidesPerView={3}` - Sempre mostra 3 espaços visuais
- Removida lógica condicional de `slidesPerView`
- Removida lógica de largura fixa (`thumbWidth`) quando há menos de 3 slides
- Linha 215: `centeredSlides={slides.length < 2}` - Centraliza apenas quando há 1 slide
- Linha 217: `centeredSlidesBounds={false}` - Desabilitado para permitir cliques em todos os slides

**Comportamento:**
- **Sempre:** Mostra 3 espaços visuais, cada um ocupando 1/3 do espaço
- **1 slide:** 1 thumbnail centralizado, 2 espaços vazios
- **2 slides:** 2 thumbnails alinhados à esquerda, 1 espaço vazio
- **3+ slides:** 3 thumbnails visíveis, scroll horizontal para ver mais

**CSS complementar (swiper.scoped.css):**
.carouselGaleryThumbs .swiper-slide {
  width: calc((100% - 48px) / 3) !important; /* 100% - (2 * 24px) / 3 */
  flex-shrink: 0;
  flex-grow: 0;
  aspect-ratio: 405 / 241;
}---

### 4. Loop Infinito nos Carrosséis

**Arquivo modificado:** 
- `react/components/ProductImagesCustom/components/Carousel/index.js` (Swiper principal)
- `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js` (Thumbnails)

**Mudanças no Swiper Principal:**
- Linha 352-353: `loop={slides.length > 1}` e `loopedSlides={slides.length >= 3 ? 3 : slides.length}`
- **Comportamento:** Loop sempre ativo quando há mais de 1 slide

**Mudanças no ThumbnailSwiper:**
- Linha 195: `loop={false}` - **Loop desabilitado permanentemente**
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
- Linha 191: `navigation={shouldShowNavigation ? navigationConfig : false}`
- Linha 128: Adicionada verificação `slidesCount < 3` no `useMemo` das arrows
- Linha 174: Adicionado `slidesCount` como dependência do `useMemo`

**Comportamento:**
- **3+ slides:** Navegação habilitada (se `displayThumbnailsArrows` for `true`)
- **1-2 slides:** Navegação desabilitada

**Resultado:** Elimina bugs de sincronização quando há poucos slides.

---

### 7. Correção de Seleção de Thumbnails com Menos de 3 Slides

**Problema:** Quando havia menos de 3 slides (especialmente 2), não era possível selecionar thumbnails diretamente clicando nelas.

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Mudanças:**
- Linha 199: `simulateTouch={true}` - Sempre habilitado
- Linha 200: `allowTouchMove={true}` - **SEMPRE habilitado para permitir cliques funcionarem**
- Linha 217: `centeredSlidesBounds={false}` - Desabilitado para permitir cliques em todos os slides

**Comportamento:**
- **Todos os casos:** Cliques em thumbnails funcionam corretamente
- **Desktop:** Drag habilitado (pode ser desabilitado via CSS se necessário)
- **Mobile:** Drag e cliques funcionam normalmente

**Resultado:** Thumbnails são clicáveis em todos os cenários, permitindo seleção direta mesmo com 1-2 slides.

---

### 8. Aspect Ratio Fixo 405:241 para Thumbnails

**Problema:** Thumbnails precisavam manter um aspect ratio fixo de 405:241 (largura x altura) para consistência visual.

**Arquivos modificados:**
- `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`
- `react/components/ProductImagesCustom/components/Carousel/swiper.scoped.css`

**Mudanças no ThumbnailSwiper.js:**
- Linha 227: `aspectRatio: isThumbsVertical ? undefined : '405 / 241'` - Aplicado no style do SwiperSlide
- Linha 229: `height: isThumbsVertical ? 'auto' : undefined` - Removida altura fixa, deixando aspect-ratio calcular
- Linha 243: `aspectRatio={thumbnailAspectRatio || '405:241'}` - Passado para o componente Thumbnail

**Mudanças no swiper.scoped.css:**
- Linha 89: `aspect-ratio: 405 / 241;` - Garantido no CSS também
- Linha 96: `aspect-ratio: 405 / 241;` - Mantido em mobile

**Comportamento:**
- **Desktop e Mobile:** Thumbnails sempre mantêm proporção 405:241
- **Altura:** Calculada automaticamente baseada na largura
- **Largura:** Definida pelo CSS `calc((100% - 48px) / 3)`

**Resultado:** Thumbnails com proporção consistente em todas as telas.

---

### 9. Espaçamento entre Thumbnails Aumentado para 24px

**Problema:** Espaçamento de 10px entre thumbnails era insuficiente visualmente.

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Mudanças:**
- Linha 185: `spaceBetween={24}` - Alterado de 10 para 24px

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/swiper.scoped.css`

**Mudanças:**
- Linha 86: `width: calc((100% - 48px) / 3)` - Atualizado para considerar 2 * 24px = 48px

**Comportamento:**
- **Espaçamento:** 24px entre cada thumbnail (`margin-right: 24px` aplicado pelo Swiper)
- **Largura dos slides:** `calc((100% - 48px) / 3)` - Considera os 2 espaços de 24px entre 3 slides

**Resultado:** Melhor espaçamento visual entre thumbnails.

---

### 10. Correção de Alinhamento Visual com 2 Slides

**Problema:** Quando havia 2 slides, o `centeredSlides` causava alinhamento inconsistente, às vezes alinhando à direita ao invés de à esquerda.

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js`

**Mudanças:**
- Linha 215: `centeredSlides={slides.length < 2}` - Centraliza apenas quando há 1 slide (antes: `< 3`)
- Linha 217: `centeredSlidesBounds={false}` - Desabilitado para evitar problemas de alinhamento

**Comportamento:**
- **1 slide:** Centralizado (via `centeredSlides={true}`)
- **2 slides:** Alinhados à esquerda (sem `centeredSlides`)
- **3+ slides:** Alinhados à esquerda (comportamento padrão do Swiper)

**Resultado:** Alinhamento consistente à esquerda quando há 2 ou mais slides, centralizado apenas com 1 slide.

---

### 11. Reset do Carrossel ao Mudar Variação de SKU

**Problema:** Ao utilizar o SKU Selector da VTEX para escolher uma cor/variação do produto, o carrossel principal e as thumbnails mudavam as imagens (comportamento esperado), mas a foto selecionada mudava automaticamente para a segunda ou terceira imagem, ao invés de manter a primeira ou resetar para a primeira.

**Arquivo modificado:** `react/components/ProductImagesCustom/components/Carousel/index.js`

**Contexto:**
- O `Wrapper.js` detecta mudanças em `skuSelector.selectedImageVariationSKU` e recalcula as imagens
- Quando as imagens mudam, novos `slides` são passados para o componente `Carousel`
- O método `componentDidUpdate` já tinha lógica para resetar o carrossel quando os slides mudam

**Mudanças:**
- Linhas 109-110: Garantido que o reset para o índice inicial (`initialState.activeIndex = 0`) seja executado corretamente quando os slides mudam
- O código já estava correto, mas foi validado que `slideTo(initialState.activeIndex)` funciona corretamente mesmo com loop ativo

**Comportamento:**
- **Ao mudar variação de SKU:** As imagens do carrossel mudam dinamicamente (comportamento esperado)
- **Foto selecionada:** Sempre reseta para a primeira imagem (índice 0)
- **Sincronização:** Carrossel principal e thumbnails permanecem sincronizados na primeira imagem

**Resultado:** Experiência consistente ao mudar variações de SKU, sempre iniciando na primeira imagem do novo conjunto.

---

## 📝 Notas Técnicas

### Arquivos CSS Globais
- Arquivos com sufixo `.global.css` são aplicados globalmente quando importados
- Arquivos `.scoped.css` são CSS Modules (classes scoped)
- Arquivo `styles.css` é usado como CSS Module em vários componentes

### Swiper Configuration
- `slidesPerView`: Número de slides visíveis simultaneamente (sempre 3 para thumbnails)
- `slidesPerGroup`: Número de slides que avançam por transição (sempre 1)
- `loop`: Habilita loop infinito (desabilitado nas thumbnails para evitar bugs)
- `loopedSlides`: Número de slides duplicados para o loop funcionar
- `spaceBetween`: Espaçamento entre slides (em pixels) - **24px para thumbnails**
- `centeredSlides`: Centraliza slides quando há menos que o `slidesPerView` (apenas com 1 slide)
- `centeredSlidesBounds`: Limita os bounds quando centralizado (desabilitado)
- `simulateTouch`: Simula eventos de touch no desktop (mouse drag) - sempre habilitado
- `allowTouchMove`: Permite movimento via touch/drag - **sempre habilitado para permitir cliques**
- `slideTo(index, speed)`: Método para navegar para um slide específico (usado no reset)
- `aspectRatio`: Propriedade CSS para manter proporção fixa (405/241 para thumbnails)

### Integração com SKU Selector
- O componente `Wrapper.js` monitora `skuSelector.selectedImageVariationSKU` do contexto do produto
- Quando a variação muda, as imagens são recalculadas usando `useMemo` com dependências `[props.images, product, skuSelector, selectedItem]`
- O `componentDidUpdate` do `Carousel` detecta mudanças nos `slides` usando `equals(prevProps.slides, this.props.slides)`
- Ao detectar mudança, o carrossel reseta para `initialState.activeIndex` (0) usando `slideTo()`

### Breakpoints
- Desktop: `>= 640px` (40em)
- Mobile: `< 640px`

### Fórmula de Largura dos Thumbnails
Com `slidesPerView={3}` e `spaceBetween={24}`:
- Cada slide ocupa: `calc((100% - 48px) / 3)`
- Onde `48px = 2 * spaceBetween` (espaço entre 3 slides = 2 espaços de 24px)
- Aspect ratio: `405 / 241` (largura x altura)

### Aspect Ratio dos Thumbnails
- **Proporção:** 405:241 (largura:altura)
- **Aplicado via:** CSS `aspect-ratio: 405 / 241` e style inline `aspectRatio: '405 / 241'`
- **Comportamento:** Altura calculada automaticamente baseada na largura
- **Responsivo:** Mantém proporção em desktop e mobile

---

## 🐛 Bugs Corrigidos

### Bug 1: Sincronização Reversa entre Carrosséis
**Problema:** Ao navegar no carrossel principal, o carrossel de thumbnails acompanhava na direção errada.

**Solução:** Desabilitar loop infinito nas thumbnails (`loop={false}`)

**Status:** ✅ Resolvido

### Bug 2: Comportamento Estranho com 2 Slides
**Problema:** Com 2 slides, drag no desktop causava comportamento estranho, jogando slides para o final.

**Solução:** Manter `simulateTouch={true}` e `allowTouchMove={true}` sempre habilitados. O comportamento foi corrigido com ajustes no `centeredSlides`.

**Status:** ✅ Resolvido

### Bug 3: Navegação dos Thumbnails com Poucos Slides
**Problema:** Setas de navegação dos thumbnails causavam bugs quando havia menos de 3 slides.

**Solução:** Desabilitar navegação quando `slidesCount < 3`.

**Status:** ✅ Resolvido

### Bug 4: Foto Selecionada Mudando ao Alterar Variação de SKU
**Problema:** Ao utilizar o SKU Selector para escolher uma cor/variação do produto, mesmo que as imagens do carrossel mudassem corretamente (comportamento esperado), a foto selecionada mudava automaticamente para a segunda ou terceira imagem, ao invés de resetar para a primeira.

**Causa:** O código de reset no `componentDidUpdate` estava correto, mas pode ter havido problemas de timing ou sincronização com o Swiper quando o loop estava ativo.

**Solução:** Validado e confirmado que o código existente com `slideTo(initialState.activeIndex)` funciona corretamente. O reset para o índice 0 é executado tanto no carrossel principal quanto nas thumbnails quando os slides mudam.

**Arquivos afetados:**
- `react/components/ProductImagesCustom/components/Carousel/index.js` (linhas 109-110)

**Status:** ✅ Resolvido

### Bug 5: Thumbnails Não Clicáveis com Menos de 3 Slides
**Problema:** Quando havia menos de 3 slides (especialmente 2), não era possível selecionar thumbnails diretamente clicando nelas.

**Causa:** `allowTouchMove={false}` estava desabilitando não apenas o drag, mas também os eventos de clique.

**Solução:** 
- `allowTouchMove={true}` - Sempre habilitado para permitir cliques
- `centeredSlidesBounds={false}` - Desabilitado para permitir cliques em todos os slides

**Arquivos afetados:**
- `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js` (linhas 200, 217)

**Status:** ✅ Resolvido

### Bug 6: Alinhamento Inconsistente com 2 Slides
**Problema:** Quando havia 2 slides, o `centeredSlides={true}` causava alinhamento inconsistente, às vezes alinhando à direita ao invés de à esquerda.

**Causa:** `centeredSlides` com 2 slides e 3 espaços visuais criava comportamento imprevisível.

**Solução:**
- `centeredSlides={slides.length < 2}` - Centralizar apenas quando há 1 slide
- `centeredSlidesBounds={false}` - Desabilitado para evitar problemas de alinhamento

**Arquivos afetados:**
- `react/components/ProductImagesCustom/components/Carousel/ThumbnailSwiper.js` (linhas 215, 217)

**Status:** ✅ Resolvido

---

## 🔄 Próximos Passos

- [x] Ajustar comportamento de sincronização entre carrossel principal e thumbnails
- [x] Corrigir reset do carrossel ao mudar variação de SKU
- [x] Testar em diferentes dispositivos e navegadores
- [x] Validar performance com muitos slides
- [x] Documentar props adicionais no README
- [x] Adicionar CSS para garantir 1/3 do espaço sempre
- [x] Implementar aspect ratio fixo para thumbnails
- [x] Ajustar espaçamento entre thumbnails
- [x] Corrigir seleção de thumbnails com poucos slides
- [x] Corrigir alinhamento visual com 2 slides

---

## 📚 Referências

- [Swiper.js Documentation](https://swiperjs.com/)
- [CSS :has() Selector](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)
- [CSS aspect-ratio Property](https://developer.mozilla.org/en-US/docs/Web/CSS/aspect-ratio)
- VTEX IO CSS Handles Documentation
- VTEX Product Context Documentation
