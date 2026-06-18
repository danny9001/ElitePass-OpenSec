# Diseño Técnico: Responsive Design Global

## Stack Actual
- **Tailwind CSS v4** con `@theme` custom en `globals.css`
- **Breakpoints predeterminados**: `sm` (640px), `md` (768px), `lg` (1024px), `xl` (1280px), `2xl` (1536px)
- **Componentes**: React 19 + Next.js 16 (App Router)

## Decisiones Técnicas

### 1. Estrategia de Breakpoints

**Usaremos breakpoints Tailwind estándar + nuevos para 1920px:**

```
- base/mobile: 360px - 639px (teléfonos)
- sm: 640px - 767px (tablets pequeños)
- md: 768px - 1023px (tablets)
- lg: 1024px - 1279px (laptops HD)
- xl: 1280px - 1535px (desktops)
- 2xl: 1536px+ (pantallas grandes 1920px+)
```

Tailwind ya soporta esto nativamente. **No se requieren cambios en tailwind.config.ts**.

### 2. Componentes Escondibles (Hide/Show)

Pattern para componentes que aparecen/desaparecen según tamaño:

```tsx
{/* Modal de Tinkaso - Solo en móvil/tablet */}
<div className="block lg:hidden">
  <button onClick={() => setTinkasoStatsModal(true)}>Ver Tinkaso Stats</button>
</div>

{/* Panel de Tinkaso - Solo en desktop grande */}
<div className="hidden lg:block">
  <TinkasoStatsPanel />
</div>
```

### 3. Layouts Responsive

**Opción A: Grid Flexible (RECOMENDADA)**
```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 2xl:grid-cols-4 gap-4">
  {items.map(item => <Card key={item.id} />)}
</div>
```

**Opción B: Flex + Wrapping**
```tsx
<div className="flex flex-wrap lg:flex-nowrap gap-4">
  <Sidebar className="w-full lg:w-64" />
  <Content className="w-full lg:flex-1" />
</div>
```

### 4. Eliminación de `max-w-*` Restrictivos

**Problema:** Muchas páginas tienen `max-w-md`, `max-w-2xl`, `max-w-screen-xl`  
**Solución:** Reemplazar con responsive widths:

```tsx
{/* ANTES - Restricción en todos los breakpoints */}
<div className="max-w-2xl mx-auto">

{/* DESPUÉS - Responsivo */}
<div className="w-full max-w-2xl lg:max-w-full 2xl:max-w-full mx-auto px-4">
```

O mejor aún:

```tsx
{/* Usar padding en lugar de max-w */}
<div className="px-4 lg:px-8">
  <div className="grid grid-cols-1 lg:grid-cols-2 2xl:grid-cols-3">
    ...
  </div>
</div>
```

### 5. Admin Panel - Caso Especial (2386 líneas)

**Arquitectura actual:** Monolítico con tabs (`usuarios | empresa | mensajes | pagos`)

**Mejora responsiva:**

```tsx
{/* Navegación - Stack vertical en móvil, horizontal en desktop */}
<div className="flex flex-col lg:flex-row gap-4">
  {/* Tabs/Navigation - Sidebar en desktop */}
  <nav className="w-full lg:w-48 flex lg:flex-col gap-1">
    {tabs.map(tab => <TabButton />)}
  </nav>
  
  {/* Contenido - Expandido */}
  <main className="w-full lg:flex-1">
    {activeTabContent}
  </main>
</div>
```

### 6. Tinkaso Stats - Transformación

**Móvil (base-lg):**
- ✅ Botón "Tinkaso Stats" que abre modal
- ✅ Modal con lista vertical de teams

**Desktop (lg+):**
- ✅ Panel lateral (grid 2 columnas)
- ✅ Mostrado siempre (no modal)
- ✅ Actualización en tiempo real con SSE

```tsx
{/* Tinkaso - Escondido en desktop */}
<button className="lg:hidden" onClick={() => setTinkasoStatsModal(true)}>
  Ver Tinkaso Stats
</button>

{/* Tinkaso - Visible en desktop */}
<div className="hidden lg:block">
  <TinkasoStatsPanel data={tinkasoStats} />
</div>
```

### 7. Componentes que Requieren Cambios

| Página | Cambio | Breakpoint |
|--------|--------|------------|
| **ranking** | Tinkaso como panel lateral | `lg:` |
| **admin** | Sidebar navigation | `lg:` |
| **partidos** | Grid 2-col en desktop | `2xl:` |
| **dashboard** | Grid 3-4 col en desktop | `lg:` |
| **perfil** | Dos columnas en desktop | `lg:` |
| **fixture** | Grid flexible en desktop | `lg:` |
| **reglas** | Max-width removido en desktop | `lg:` |

---

## Implementación por Fases

### Fase 1: Funcionalidad Base
1. Extraer `TinkasoStatsPanel` como componente reutilizable
2. Aplicar `lg:hidden` / `hidden lg:block` en ranking
3. Testear en móvil (360px) y desktop (1920px)

### Fase 2: Admin Panel
1. Refactorizar navegación a sidebar
2. Aplicar layout flex con `lg:flex-row`
3. Mantener tabs en móvil, sidebar en desktop

### Fase 3: Otras Páginas
1. Actualizar grid layouts
2. Remover `max-w-*` restrictivos
3. Agregar padding responsivo

### Fase 4: Testing
1. Validar en breakpoints clave (360, 768, 1024, 1920)
2. Revisar scroll behavior
3. Testing en navegadores reales

---

## Riesgos y Mitigación

| Riesgo | Mitigación |
|--------|-----------|
| Layout roto en algunos breakpoints | Testear en DevTools con viewport simulado |
| Performance (demasiados `lg:`, `2xl:`) | Usar CSS variables, no inline Tailwind |
| Modales ocultos pero aún en DOM | Usar `display: none` (Tailwind lo maneja) |
| Compatibilidad con navegadores viejos | Tailwind v4 soporta IE11 via fallbacks |

---

## Herramientas de Testing

- **Chrome DevTools**: Device Toolbar para simular breakpoints
- **Responsive Design Checker** (online)
- **Lighthouse**: Performance en mobile vs desktop

---

## Archivos a Crear/Modificar

### Nuevos Archivos
- `src/components/TinkasoStatsPanel.tsx` - Componente extraído

### Archivos Modificados
- `src/app/(app)/ranking/page.tsx`
- `src/app/(app)/admin/page.tsx`
- `src/app/(app)/dashboard/page.tsx`
- `src/app/(app)/partidos/page.tsx`
- `src/app/(app)/perfil/page.tsx`
- `src/app/(app)/fixture/page.tsx`
- `src/app/(app)/reglas/page.tsx`
- `src/app/globals.css` (si se requieren custom utilities)

---

## Métricas de Éxito

✅ **Móvil**: Layout compacto, navegación clara  
✅ **Desktop**: Componentes aprovechan horizontal space  
✅ **1920px**: Uso de >70% del viewport ancho  
✅ **Performance**: No degradación en Core Web Vitals  
✅ **Funcionalidad**: 100% features en todos los breakpoints
