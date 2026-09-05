# Checklist Personalizada — Power-Up de Trello

Power-Up que reemplaza el flujo de checklist por uno propio, con **checklists
múltiples por tarjeta, subtareas anidadas dentro de cada ítem, y plantillas
reutilizables** a nivel de tablero:

- **Botón en cada tarjeta** → abre el administrador de checklists de esa tarjeta.
- Una tarjeta puede tener **varias checklists** (igual que en Trello nativo:
  "AYSA", "AYSA 2", etc.), cada una con su propia barra de progreso.
- **Cada ítem se puede desplegar (▸) y agregarle subtareas propias.** El ítem
  padre se marca automáticamente como hecho cuando todas sus subtareas están
  completas (su checkbox queda deshabilitado mientras tenga subtareas, porque
  se calcula solo).
- **Badge en el frente de la tarjeta** → contador `hecho/total` sumando todas
  las checklists y subtareas de la tarjeta.
- **Badge en el detalle de la tarjeta** → porcentaje completado (clickeable).
- **Botón en el tablero** → administrador de plantillas (crear/eliminar).
- Desde una tarjeta se puede **crear una checklist nueva a partir de una
  plantilla** (trae sus ítems ya cargados, sin subtareas — se agregan después
  desplegando cada ítem).

Los datos se guardan con el almacenamiento propio del Power-Up
(`t.get` / `t.set`), no en la checklist nativa de Trello — por eso no hace
falta ningún backend propio.

## Estructura de archivos

```
trello-checklist-powerup/
├── index.html       ← conector: registra las capacidades ante Trello
├── checklist.html    ← popup de la tarjeta (ver/editar ítems)
├── templates.html    ← popup del tablero (crear/eliminar plantillas)
├── style.css          ← estilos compartidos
└── icons/
    └── checklist-icon.svg
```

## 1. Publicar en GitHub Pages

1. Creá un repositorio nuevo en GitHub (puede ser público o privado si tenés
   GitHub Pro/Team; Pages gratuito con dominio `github.io` requiere que el
   repo sea público).
2. Subí **todo el contenido** de esta carpeta a la raíz del repo:
   ```bash
   cd trello-checklist-powerup
   git init
   git add .
   git commit -m "Power-Up: checklist personalizada"
   git branch -M main
   git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
   git push -u origin main
   ```
3. En GitHub: **Settings → Pages → Source: Deploy from a branch → Branch:
   `main` / `root`** → Guardar.
4. Esperá 1–2 minutos. Tu Power-Up quedará servido en:
   ```
   https://TU_USUARIO.github.io/TU_REPO/
   ```
   Verificá que `https://TU_USUARIO.github.io/TU_REPO/index.html` cargue sin
   error 404.

> Nota: como todo corre client-side vía HTTPS servido por GitHub Pages,
> cumple el requisito de Trello de que el conector esté en una URL segura
> pública.

## 2. Registrar el Power-Up en Trello

1. Andá a **[trello.com/power-ups/admin](https://trello.com/power-ups/admin)**
   con tu cuenta.
2. **Nuevo Power-Up / integración** → completá nombre, workspace dueño, email
   de soporte, etc.
3. En **"URL del iframe conector"** pegá:
   ```
   https://TU_USUARIO.github.io/TU_REPO/index.html
   ```
4. En la pestaña **"Capacidades"**, habilitá:
   - `card-badges`
   - `card-detail-badges`
   - `card-buttons`
   - `board-buttons`
   (cada una ya tiene su código implementado en `index.html`; sólo hay que
   activarlas en el panel).
5. Guardá los cambios.

## 3. Habilitarlo en un tablero

1. Abrí el tablero donde lo querés usar → menú **"Mostrar menú" → Power-Ups**.
2. Buscá tu Power-Up en la pestaña **"Personalizado"** (te va a aparecer
   porque lo registraste en tu workspace) y hacé clic en **Agregar**.
3. Ya deberías ver el botón **"Plantillas de checklist"** arriba del tablero,
   y el botón **"Checklist personalizada"** al abrir cualquier tarjeta.

## Cómo se guardan los datos

| Dato | Clave | Alcance (`scope`) | Visibilidad |
|---|---|---|---|
| Checklists de una tarjeta | `checklists` | `card` | Compartido (`shared`) — visible para todos los miembros del tablero |
| Plantillas | `templates` | `board` | Compartido (`shared`) — un set de plantillas por tablero |

Estructura de `checklists` (array, una entrada por checklist de la tarjeta):

```js
[
  {
    id: "abc123",
    name: "AYSA",
    items: [
      {
        id: "def456",
        name: "Hacer Base de pagos provisoria",
        checked: false,      // se recalcula solo si tiene subitems
        subitems: [
          { id: "ghi789", name: "Revisar cancelados", checked: false },
          { id: "jkl012", name: "Confirmar sin cierre", checked: true }
        ]
      }
    ]
  }
]
```

Si más adelante querés que las plantillas sean privadas por usuario en lugar
de compartidas por tablero, hay que cambiar `'shared'` por `'private'` en las
llamadas `t.get('board', 'shared', ...)` de `templates.html` e
`index.html`.

## Personalización rápida

- **Cambiar colores:** editá las variables al principio de `style.css`
  (`--trello-blue`, `--green`, etc.).
- **Agregar fecha límite por ítem o subtarea:** en `checklist.html`, sumale
  `{dueDate}` al objeto correspondiente (`item` o `subitem`) y renderizá un
  `<input type="date">` extra en `buildItemRow` / `buildSubitemRow`.
- **Permitir subtareas de subtareas:** hoy el anidado tiene un solo nivel
  (ítem → subtareas). Para un segundo nivel, `buildSubitemRow` tendría que
  reutilizar la misma lógica de expandir/agregar que usa `buildItemRow`.
- **Mover la tarjeta automáticamente al 100%:** en `index.html`, dentro de
  `'card-badges'`, cuando `p.done === p.total` podés llamar a la API REST de
  Trello (`t.getRestApi()...`) para mover la tarjeta a otra lista.
