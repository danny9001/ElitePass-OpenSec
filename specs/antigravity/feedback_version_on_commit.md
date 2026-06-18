---
name: feedback-version-on-commit
description: En cada commit se debe subir la versión en package.json con el formato 1.1.XXX donde XXX es el número de commit
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8fa2753e-733d-426c-8c1f-d867ef103c6c
---

En cada commit se debe actualizar la versión en `package.json` con el formato `1.1.XXX`, donde `XXX` es el número secuencial del commit (1, 2, 3...).

**Why:** El usuario quiere trazabilidad directa entre número de versión y número de commit para identificar fácilmente qué cambios corresponden a qué versión.

**How to apply:** Antes de hacer `git commit`, contar los commits existentes (`git rev-list --count HEAD`) y sumar 1, luego actualizar `package.json` con `"version": "1.1.N"`. Editar el archivo con la herramienta Edit antes de stagear y commitear.
