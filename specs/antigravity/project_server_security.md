---
name: project-server-security
description: Configuración de seguridad del servidor de producción — puertos SSH y firewall
metadata: 
  node_type: memory
  type: project
  originSessionId: 02f7ca94-307d-4e17-80f2-8a098c4040ea
---

El servidor usa el puerto 5001 (o 5011) para SSH, NO el puerto 22 estándar. El puerto 22 debe estar **cerrado** en UFW por seguridad.

**Why:** El usuario cierra el puerto 22 por seguridad para evitar ataques de fuerza bruta y escaneos automáticos al puerto SSH estándar.

**How to apply:** Al revisar o modificar reglas UFW, verificar que el puerto 22 no esté abierto. El acceso SSH es vía puerto 5001 o 5011. Si el puerto 22 aparece abierto, señalarlo al usuario como un riesgo de seguridad.
