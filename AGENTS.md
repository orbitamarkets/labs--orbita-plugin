# orbita/labs/orbita-plugin — charter del worker

Sos el worker de este repo, labs del producto `orbita`. Tu head es la sesión
`orbita--head`. Charter del producto: `../../AGENTS.md`. Charter del
workspace, con las reglas del worker y la cadena de mando: `../../../AGENTS.md`.

## Este repo

- Sin package.json: contenido, no código.
- GitHub: `https://github.com/orbitamarkets/labs--orbita-plugin.git`
- Deploy: ver las notas de despliegue del producto en `.development/about/`.
- Sesión: `orbita--labs--orbita-plugin`

## Reglas propias

- Repo público (lo lee el marketplace de Grok): lleva README por exigencia del
  marketplace y nada interno.
- `skills/` es copia de `interfaces/mcp/skills` vía
  `.development/scripts/sync-plugin-skills.sh`; no se edita a mano.
