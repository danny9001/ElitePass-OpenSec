# Design: Página de Edición de Empresa con Búsqueda y Gestión de Usuarios

## Arquitectura de la Solución

### Componentes y Rutas
- **Página de Edición (`/dashboard/empresas/[id]/editar/page.tsx`):**
  - Carga del lado del servidor (React Server Component) utilizando datos únicos de `db.empresa`, la lista completa de usuarios `db.user` ordenados alfabéticamente (con su clasificación y membresías de empresas), y todas las empresas del sistema `db.empresa`.
  - Pasa esta información al componente cliente `EditarEmpresaForm`.
  
- **Componente Cliente (`EditarEmpresaForm.tsx`):**
  - **Pestañas (Tabs):**
    - **Datos Generales:** Formulario estándar de la empresa con campos como nombre, NIT, teléfono, localización y colores primario/secundario.
    - **Usuarios / Miembros:** Interfaz interactiva dividida en dos columnas para gestionar la asignación de miembros.
  - **Gestión de Selección de Usuarios:**
    - Estado reactivo local para los `selectedPersonas` (un arreglo de IDs de usuarios seleccionados).
    - Los usuarios asignados inicialmente se cargan de `empresa.miembros`.
    - La interfaz agrega o remueve del arreglo de IDs reactivamente.
  - **Buscador Integrado:**
    - Filtro local en el cliente para la búsqueda por nombre completo (`name` + `apellido`) y correo electrónico (`email`).
    - Actualización en tiempo real sin recargar la página.
  - **Filtro Avanzado por Empresa en "No Pertenecen":**
    - Selector dropdown que permite filtrar la lista de usuarios no seleccionados según:
      - Todas las empresas (comportamiento por defecto).
      - Sin empresa asignada (usuarios que no tienen ninguna membresía activa).
      - Una empresa específica (usuarios asignados a dicha empresa).
  - **Identificación de Clasificación (Tratamiento como Externos):**
    - Se visualiza de manera explícita el tipo de cuenta (`accountType`) del usuario al lado de su nombre.
    - Si el usuario está clasificado como `USER` (Usuario estándar) o `CLIENT` (Cliente), se añade la etiqueta `(Externo)` al lado, reflejando claramente que serán tratados como usuarios externos en los flujos de inicio de sesión SSO de las aplicaciones satélite (reservas, POS, etc.).
  - **Columnas de Gestión (Dos Columnas):**
    - **Pertenecen a la empresa:** Muestra usuarios seleccionados que coinciden con el filtro de búsqueda. Botón "Remover".
    - **No pertenecen a la empresa:** Muestra usuarios no seleccionados que coinciden con el filtro de búsqueda y el filtro de empresa seleccionada. Botón "Agregar".

### Compatibilidad con Backend (`updateEmpresa` Action)
El backend procesa la petición a través de la Server Action `updateEmpresa(id, formData)`.
El backend obtiene la lista de IDs de usuario mediante:
```typescript
const personaIds = formData.getAll("personaIds") as string[];
```
Luego elimina todos los registros en `PersonaEmpresa` para esa empresa e inserta los nuevos IDs.
Para mantener compatibilidad absoluta sin alterar la firma ni la lógica del backend:
1. Renderizamos inputs ocultos `<input type="hidden" name="personaIds" value={id} />` para cada ID en `selectedPersonas`.
2. En `handleSubmit`, interceptamos el submit para limpiar el `personaIds` del `FormData` y agregar manualmente cada ID seleccionado, asegurando que se capture la selección reactiva del cliente.
