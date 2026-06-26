# Design: Premium File Upload Components

## UI/UX Design Goals & Aesthetics
We aim for a modern, fluid, and premium experience matching SaaS platforms like Vercel, Stripe, or Linear.

### 1. Color Palette (HSL & Tailwind)
We leverage custom HSL variables to support a dynamic, modern appearance in both Light and Dark modes.

| Concept | HSL Values (Dark Mode) | HSL Values (Light Mode) | Tailwind Equivalent |
| :--- | :--- | :--- | :--- |
| **Brand Accent** | `262 83% 58%` (Indigo-Violet) | `262 83% 58%` | `bg-indigo-600` / `from-purple-500 to-indigo-500` |
| **Success State**| `142 70% 45%` (Emerald) | `142 76% 36%` | `text-emerald-500` / `border-emerald-500/30` |
| **Success BG**  | `142 70% 8%` | `142 70% 97%` | `bg-emerald-500/10` / `bg-emerald-50` |
| **Error State**  | `350 89% 60%` (Rose) | `350 89% 45%` | `text-rose-500` / `border-rose-500/30` |
| **Error BG**    | `350 89% 8%` | `350 89% 96%` | `bg-rose-500/10` / `bg-rose-50` |
| **Background**   | `222 47% 11%` (Slate-950) | `0 0% 100%` (White) | `bg-slate-950` / `bg-white` |
| **Card / Border**| `222 47% 17%` / `222 47% 25%`| `220 13% 91%` / `220 13% 85%` | `bg-slate-900` / `border-slate-800` |

### 2. State & Visual Feedback Spec

#### A. Idle / Empty Dropzone
*   **Aesthetics**: Dashed border with custom dash array (`border-dashed border-2`), smooth border transition (`transition-all duration-300`).
*   **Interactions**: Hovering zooms the central icon slightly (`scale-110 duration-200`) and shifts border color from slate to the brand accent (indigo).
*   **Details**: Format list (e.g., `PNG, JPG, SVG up to 2MB` or `PDF, PNG, JPG up to 10MB`) rendered in a soft, low-contrast text color.

#### B. Active Progress (Uploading)
*   **Text Indicator**: Displays `"Subiendo..."` along with a crisp percentage indicator (e.g., `45%`).
*   **Progress Bar Track**: High-precision track (`h-1.5 w-full bg-slate-800 rounded-full overflow-hidden`).
*   **Progress Bar Fill**: Smooth gradient (`bg-gradient-to-r from-purple-500 to-indigo-500`) with an overlaying shimmer effect using a CSS animation keyframe. Width transition uses `transition-all duration-300 ease-out`.
*   **Cancel Trigger**: Small, sleek `X` button on the far right (or a text button `"Cancelar"`) that is highly responsive.

#### C. Success State
*   **Container**: Border changes to soft emerald (`border-emerald-500/30` or solid `border-emerald-500` for light border), background transitions to a very subtle translucent green.
*   **Iconography**: A green check circle (`CheckCircle2` from Lucide) with a scale spring-in animation (`scale-0` to `scale-100` with `ease-out`).
*   **Actions**: File metadata (name, size) shown with a `"Reemplazar"` or `"Eliminar"` button.

#### D. Error State
*   **Container**: Border shifts to soft rose (`border-rose-500/30`), background shifts to a very subtle translucent red.
*   **Iconography**: Alert circle (`AlertCircle` from Lucide) in warning rose color.
*   **Description**: Explicit error messages like `"El archivo excede el tamaño máximo de 10MB"` or `"Formato de archivo no compatible"`.
*   **Actions**: A highly visible `"Reintentar"` (Retry) text button or icon button that resets the state and triggers the file browser again.

### 3. Component Context Configurations

#### Component A: Branding Asset Uploader (Logo / Icon)
*   **Layout**: Compact, centered, square (1:1 aspect ratio with optional circular border preview for icons) or horizontal (16:9 for logos).
*   **Placement**: Embedded directly in the Organization Settings form.
*   **Interactions**: Overlay hover effect showing `"Cambiar imagen"` on top of the current logo when uploaded.

#### Component B: Payment Voucher Uploader (Document)
*   **Layout**: Wide, card-based, with drag-and-drop zone.
*   **Interactions**: Displays file icon (e.g., PDF or Image icon) with download and full-screen view (lightbox) capability once uploaded.
*   **Security scanner step (Optional/Premium)**: An extra status step showing `"Analizando archivo..."` using a scanning animation before completion, giving it a secure, high-end feel.

### 4. Coordination with Form Buttons
*   **Sync State**: The parent form maintains an `isUploading` boolean flag.
*   **State Propagation**: Whenever `isUploading` is true, the primary submit/save button:
    1.  Sets `disabled={true}` to block clicks and adds `cursor-not-allowed opacity-50` styles.
    2.  Renders a loading spinner (`animate-spin` on a Lucide `Loader2` icon).
    3.  Replaces the button label with `"Espere, subiendo archivos..."` or similar dynamic text.

## Root Cause Analysis & Technical Solutions

### 1. Root Cause: Why Logo/Icon Updates Do Not Persist on Save (Form Overwrite)
In `organization-form-drawer.tsx`, the form utilizes React Hook Form to manage state. The `logo` property exists in the form schema (`organizationFormSchema`).
- **The Flow**:
  1. The form initializes `logo` with the organization's existing logo URL.
  2. The user uploads a new logo file, which triggers `uploadOrganizationAsset`, immediately updating the database record with the new URL.
  3. However, the form state's `logo` field remains unchanged, containing the old URL.
  4. When the user clicks the "Actualizar" (Save) button, the form submit handler submits the current form values, which still holds the old URL, overwriting the new URL in the database.
- **The Solution**:
  Once a file upload successfully resolves with a CDN URL, the client-side component must sync this new URL back to the form state:
  ```typescript
  if (key === "logo" && cdnUrl) {
    form.setValue("logo", cdnUrl);
  }
  ```

### 2. Error Handling (Try-Catch-Finally) on Client Components
To prevent the upload state from locking up on network or execution errors:
- We wrap all upload server action/API requests in a `try-catch-finally` block.
- Reset the active uploading key/asset to `null` and revoke any temporary browser object URLs in the `finally` block to prevent UI lockup and memory leaks.
- Clear file input references to allow re-selection of the same file if an error occurred.

### 3. Progress Bar Mechanism: Simulated vs Real XHR
- **Simulated Progress Bar (Selected)**: Since standard Server Actions do not expose XHR upload progress directly without custom routing, a simulated progress animation is used.
- **Mechanism**:
  - Starts at `0%`.
  - Increments asymptotically: moves fast initially (up to `30%`), then slows down (up to `88%`), and crawls towards `95%` using an interval.
  - Instantly jumps to `100%` inside the `finally` block when the promise resolves.
  - Clears after `500ms` to transition back to idle.
  - This provides smooth visual feedback with minimum impact on backend infrastructure.

### 4. Form Locking Coordination
- Expose an optional `onUploadingChange?: (isUploading: boolean) => void` callback in child upload components (e.g. `PaymentVoucherUpload`).
- In parent components, capture this state to disable any save/submit buttons, checkboxes, or actions that could trigger data submission during an active file upload.

