# Design & Technical Comparison: ElitePass POS vs. InvenTree

## Architecture Comparison

| Dimension | ElitePass POS | InvenTree |
|---|---|---|
| **Tech Stack** | Node.js 24 + Express 5 + Prisma ORM + React 18 | Python + Django + DRF + React |
| **Database** | PostgreSQL 16 (optimized for multi-tenant PGBouncer) | PostgreSQL / MySQL / MariaDB / SQLite |
| **Real-time** | Redis 7 + Socket.io (push updates of stock to POS/Dashboard) | AJAX / REST Polling (native WebSocket support is limited) |
| **Inventory Model** | **Event-based / Active session**: Stock is bound to `Evento` and `Barra` | **Location-based**: Stock resides in nested `StockLocation` structures |
| **Tenancy** | Native Multi-tenant (`Empresa` relation on almost all models) | Single-tenant (can be multi-instanced but not natively multi-tenant) |

## Comparison of Key Concepts & Domain Models

### 1. Stock Tracking and Locations
- **InvenTree**: Uses a hierarchical tree of `StockLocation` (e.g., `Main Warehouse -> Shelf A -> Box 3`). Stock items can be serialized or batched.
- **ElitePass**: Stock is strictly partitioned by `Evento` and location (`StockAlmacen` for the central warehouse of the event, and `StockBarra` for specific bars). It has simple stock numbers, not nested locations.

### 2. Product Structure & BOM
- **InvenTree**: Models `Part` with boolean flags (`assembly`, `trackable`, `purchasable`, `salable`). Supports multi-level Bill of Materials (BOM) where an assembly can contain other assemblies.
- **ElitePass**: Models `Producto` with flat categories, specific `UnidadProducto` (BOTELLA, LATA, etc.), and `PresentacionProducto` (e.g., Bottle vs Shot/Vaso with `factorDescuento`). Supports flat `Receta` for cocktails/combos (made of ingredients). No nested recipes.

### 3. Procurement & Purchases
- **InvenTree**: Detailed purchasing workflows. Links a `Part` to `SupplierPart` (mapping internal codes to supplier SKU, price breaks, packaging). Tracks purchase orders through draft, placed, received.
- **ElitePass**: Simple `Proveedor` linked to `CompraAlmacen` and `CompraEspecial`. It registers stock additions and records vouchers, but lacks complex supplier SKU mapping, price-break tables, or procurement lifecycle states.

### 4. Manufacturing vs. Real-time POS
- **InvenTree**: Focuses on **Build Orders** (manufacturing). Stock is allocated to a build, then consumed, producing a new stock item of the assembly.
- **ElitePass**: Focuses on **Point of Sale (POS)**. When a cocktail/combo is sold, ingredients are dynamically decremented using the recipe definition (`factorDescuento` or ounces) in a single atomic transaction.

## Core Maturity Gaps in ElitePass (Compared to InvenTree)

1. **Stock Serialization & Batching (Lotes)**: InvenTree tracks specific batches of parts (with individual expiry, batch numbers, serials). ElitePass only tracks totals per product/event, with simple expiration dates on purchases.
2. **Supplier Part Mapping**: InvenTree can map one internal product to multiple suppliers with different supplier part numbers. ElitePass assumes a 1:1 relationship or generic text for supplier purchase records.
3. **Multi-level Assemblies (Nested Recipes)**: InvenTree allows nesting assemblies. ElitePass recipes are strictly flat (ingredients must be base products).
4. **Hierarchical Warehouse Locations**: ElitePass is limited to Central Almacen and Barras. It cannot model complex warehouse sectors.
5. **Plugin/Extension System**: InvenTree has a modular plugin system for third-party integrations (ERP, labels, suppliers). ElitePass is monolithic.
