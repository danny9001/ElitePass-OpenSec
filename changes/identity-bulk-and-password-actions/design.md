# Design: User Password Management and Bulk Actions in Identity

## 1. Technical Design for Actions

We will add three server actions to `/home/soporte/elitepass-identity/src/lib/actions.ts`:

### A. Individual Password Update
```typescript
export async function updatePersonaPassword(userId: string, formData: FormData) {
  const session = await requireAdmin();
  const password = formData.get("password") as string;
  if (!password || password.trim().length < 8) {
    throw new Error("La contraseña debe tener al menos 8 caracteres.");
  }
  
  const user = await db.user.findUnique({
    where: { id: userId },
    select: { email: true }
  });
  if (!user) throw new Error("Usuario no encontrado");

  const salt = await bcrypt.genSalt(12);
  const passwordHash = await bcrypt.hash(password, salt);

  const account = await db.account.findFirst({
    where: { userId, providerId: "credential" },
  });

  if (account) {
    await db.account.update({
      where: { id: account.id },
      data: {
        password: passwordHash,
        updatedAt: new Date(),
      },
    });
  } else {
    await db.account.create({
      data: {
        id: createId(),
        accountId: user.email,
        providerId: "credential",
        userId,
        password: passwordHash,
        createdAt: new Date(),
        updatedAt: new Date(),
      },
    });
  }

  await logAudit({
    evento: "user_password_changed",
    userId: session.user.id,
    targetId: userId,
    metadata: { email: user.email },
  });

  revalidatePath("/dashboard/personas");
  revalidatePath(`/dashboard/personas/${userId}`);
}
```

### B. Bulk Password Update
```typescript
export async function bulkUpdatePersonaPasswords(userIds: string[], password?: string) {
  const session = await requireAdmin();
  if (!userIds || userIds.length === 0) return;
  if (!password || password.trim().length < 8) {
    throw new Error("La contraseña debe tener al menos 8 caracteres.");
  }

  const salt = await bcrypt.genSalt(12);
  const passwordHash = await bcrypt.hash(password, salt);

  // Run in a single transaction
  await db.$transaction(async (tx) => {
    for (const userId of userIds) {
      const user = await tx.user.findUnique({
        where: { id: userId },
        select: { email: true }
      });
      if (!user) continue;

      const account = await tx.account.findFirst({
        where: { userId, providerId: "credential" },
      });

      if (account) {
        await tx.account.update({
          where: { id: account.id },
          data: {
            password: passwordHash,
            updatedAt: new Date(),
          },
        });
      } else {
        await tx.account.create({
          data: {
            id: createId(),
            accountId: user.email,
            providerId: "credential",
            userId,
            password: passwordHash,
            createdAt: new Date(),
            updatedAt: new Date(),
          },
        });
      }
    }
  });

  await logAudit({
    evento: "bulk_user_password_changed",
    userId: session.user.id,
    metadata: { count: userIds.length, userIds },
  });

  revalidatePath("/dashboard/personas");
}
```

### C. Bulk User Deletion
```typescript
export async function bulkDeletePersonas(userIds: string[]) {
  const session = await requireAdmin();
  if (!userIds || userIds.length === 0) return;

  // Safeguard: Do not allow an admin to bulk delete themselves if they are in the list
  const filteredIds = userIds.filter(id => id !== session.user.id);
  if (filteredIds.length === 0) {
    throw new Error("No puedes eliminar tu propio usuario en una acción en bulk.");
  }

  await db.user.deleteMany({
    where: {
      id: { in: filteredIds }
    }
  });

  await logAudit({
    evento: "bulk_user_deleted",
    userId: session.user.id,
    metadata: { count: filteredIds.length, userIds: filteredIds },
  });

  revalidatePath("/dashboard/personas");
}
```

## 2. UI / UX Design

### A. Individual Password Change Form Modal
- Add a new "Key/Lock" icon button next to "Editar" in user actions in `PersonasTable` and `PersonasList` card view.
- Introduce a modal with a simple form asking for:
  - "Nueva contraseña" (New Password) input.
  - Submit button: triggers `updatePersonaPassword(userId, formData)` inside a React Transition or handling loading state.
- Add a similar "Cambiar contraseña" button next to user info in `PersonaDetailPage` (`/dashboard/personas/[id]`).

### B. Bulk Actions Interface
- Add checkboxes `[ ]` to each table row in `PersonasTable` (Desktop view) and the table header `[ ]` to select/deselect all rows.
- A floating or top action bar will appear if `selectedIds.length > 0`.
- The action bar will show:
  - "X usuarios seleccionados"
  - Button "Cambiar contraseña en bulk" (opens bulk password modal).
  - Button "Eliminar en bulk" (opens confirmation modal, turns red).
  - Button "Cancelar selección" (clears checkbox states).
