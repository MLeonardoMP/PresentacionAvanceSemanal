---
name: preview-slide
description: Previsualiza y analiza el slide HTML con Playwright MCP. Levanta servidor local, toma screenshot a 1000x580, mide métricas de layout y reporta problemas visuales. Úsalo cada vez que hagas cambios al slide antes de commitear.
---

# Preview & Análisis del Slide con Playwright MCP

## Pasos obligatorios

### 1. Levantar servidor local (si no está corriendo)
```bash
npx http-server . -p 8787 &
sleep 2
```

### 2. Navegar y capturar a resolución del slide
Usa las herramientas de Playwright MCP en este orden exacto:

1. `browser_navigate` → `http://localhost:8787/`
2. `browser_resize` → width: 1000, height: 580
3. `browser_wait_for` → time: 0.7  (espera animaciones)
4. `browser_take_screenshot` → type: png

### 3. Verificar métricas críticas del DOM
Ejecuta con `browser_evaluate`:

```javascript
() => {
  const grid = document.querySelector('.grid');
  const footer = document.querySelector('.footer-bar');
  const cards = document.querySelectorAll('.card');
  const cardInner = document.querySelector('.card-inner');
  return {
    gapGridFooter: Math.round(footer.getBoundingClientRect().top - grid.getBoundingClientRect().bottom),
    footerHeight: footer.getBoundingClientRect().height,
    footerSegments: document.querySelectorAll('.footer-bar > div').length,
    cardHeights: Array.from(cards).map(c => Math.round(c.getBoundingClientRect().height)),
    cardInnerOverflow: Math.round(cardInner.scrollHeight - cardInner.clientHeight),
    footerBorderRadius: getComputedStyle(footer).borderRadius,
    gridOverflow: Math.round(grid.getBoundingClientRect().bottom - document.querySelector('.content').getBoundingClientRect().bottom)
  };
}
```

### 4. Checklist de validación

Reporta el estado de cada punto:

| Check | Valor esperado | Resultado |
|---|---|---|
| `gapGridFooter` | ≥ 12px | ? |
| `footerHeight` | 10px | ? |
| `footerSegments` | 3 | ? |
| `cardHeights` | todos iguales | ? |
| `cardInnerOverflow` | 0px | ? |
| `footerBorderRadius` | `0px 0px 16px 16px` | ? |
| `gridOverflow` | ≤ 0px | ? |

### 5. También verificar visualmente en el screenshot:
- [ ] Esquinas inferiores redondeadas (no cortadas)
- [ ] Footer con 3 colores: Azul / Magenta / Verde
- [ ] Texto de todos los ítems completamente visible (sin recorte)
- [ ] Separación visible entre tarjetas y footer
- [ ] Tarjetas de igual altura en ambas filas
- [ ] Título con fuente Space Grotesk (más bold/display)
- [ ] Fecha dinámica en el badge (no hardcodeada)

### 6. Si se detectan problemas
- **gridOverflow > 0**: el grid desborda → revisar `height: 0` en `.grid`
- **gapGridFooter < 12**: poca separación → ajustar `padding-bottom` en `.content`
- **cardInnerOverflow > 0**: texto recortado → reducir padding/font-size en `.card-inner`
- **footerBorderRadius incorrecto**: esquinas cortadas → verificar `border-radius: 0 0 16px 16px` en `.footer-bar`

## Uso típico
Después de editar `index.html`, ejecuta `/preview-slide` para validar antes de hacer commit.
