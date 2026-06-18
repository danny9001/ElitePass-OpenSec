# Design: Sorting and Filtering Modal for Identity Personas

## UI Component Architecture
We will introduce a client component `src/components/PersonasFilters.tsx` that replaces the current search/filter `<form>` in `src/app/dashboard/personas/page.tsx`.

### State & Next.js Navigation
The component will read current query parameters from `useSearchParams()`.
When filters are applied:
1. Gather values from form inputs.
2. Build a new `URLSearchParams` object.
3. Keep the current search query `q` if present.
4. Navigate to `/dashboard/personas?` with the new query string using `router.push()`.

### Layout
- **Desktop/Mobile main row**: Search text input + "Filtrar y Ordenar" button (with `SlidersHorizontal` icon) + "Buscar" button.
- **Filter Modal**: Using the existing `<Modal>` component, with sections for:
  - **Filtrar por**:
    - Empresa (combobox/select)
    - Aplicación (select)
    - Tipo de Usuario (select: USER, CLIENT, STAFF, EXTERNAL)
    - Fecha de registro (Date range inputs: Desde / Hasta)
  - **Ordenar por**:
    - Campo (select: Fecha de registro, Nombre, Correo)
    - Orden (select/toggle: Ascendente, Descendente)
- **Active filters row**: Under the search input, display badges for each active filter with an 'x' to clear it, and a "Limpiar todo" button.

## Prisma Database Queries
In `src/app/dashboard/personas/page.tsx`, we will update the parameters query logic.

### Combine memberships filter logic
```typescript
const where: any = {};

if (q) {
  where.OR = [
    { name: { contains: q, mode: "insensitive" } },
    { email: { contains: q, mode: "insensitive" } },
    { ci: { contains: q, mode: "insensitive" } },
  ];
}

if (empresaFilter || appFilter) {
  const empresaConditions: any = {};
  if (empresaFilter) {
    empresaConditions.empresaId = empresaFilter;
  }
  if (appFilter) {
    empresaConditions.empresa = {
      licencias: {
        some: {
          appId: appFilter,
          activa: true,
        }
      }
    };
  }
  where.empresaMemberships = {
    some: empresaConditions
  };
}

if (tipoFilter) {
  where.accountType = tipoFilter;
}

if (dateFrom || dateTo) {
  where.createdAt = {};
  if (dateFrom) {
    where.createdAt.gte = new Date(dateFrom);
  }
  if (dateTo) {
    const end = new Date(dateTo);
    end.setHours(23, 59, 59, 999);
    where.createdAt.lte = end;
  }
}
```

### Ordering logic
```typescript
let orderBy: any = { createdAt: "desc" };
if (sortBy && ["createdAt", "name", "email"].includes(sortBy)) {
  const order = sortOrder === "asc" ? "asc" : "desc";
  orderBy = { [sortBy]: order };
}
```
