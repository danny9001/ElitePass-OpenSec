# Proposal: Página de Edición de Empresa con Búsqueda y Gestión de Usuarios

## Business Intent
El objetivo es proveer una nueva página de edición dedicada para empresas en `elitepass-identity` que reemplace el formulario modal anterior. Esta página debe permitir la edición de datos generales de la empresa y una gestión avanzada de la asignación de usuarios (quién pertenece y quién no) mediante una interfaz interactiva de búsqueda (por nombre o correo) y agregación/remoción inmediata.

## Scope
1. **Redirección de Edición:** Reemplazar el botón de edición en modal en `EmpresaCard` para que redirija a una nueva página en `/dashboard/empresas/[id]/editar`.
2. **Página de Edición y Componente:** Crear la ruta del servidor `/dashboard/empresas/[id]/editar/page.tsx` para cargar los datos de la empresa y de todos los usuarios del sistema.
3. **Formulario Interactivo (EditarEmpresaForm):** Crear un componente cliente con pestañas (General y Usuarios) que contenga la lógica de búsqueda de personas por correo o nombre y la clasificación interactiva (Pertenecen vs No pertenecen) con inputs ocultos para el envío compatible de `personaIds`.
