# Plantilla de descripción de Pull Request

Usen esta estructura al abrir cada PR en GitHub (feature u hotfix), pegándola en el campo de descripción.

---

## Tipo de cambio
- [ ] Feature (nueva funcionalidad)
- [ ] Hotfix (corrección urgente)

## Descripción
[Qué se hizo y por qué. Ejemplo: "Se agregó el endpoint GET /usuarios/{id} para consultar un usuario por su identificador."]

## Rama origen → destino
`feature/nombre-cambio` → `develop`
(o para hotfix: `hotfix/nombre-cambio` → `main`)

## Cómo probarlo
[Pasos breves para que el revisor pruebe el cambio.]

## Checklist
- [ ] Commits siguen la convención definida en el README.
- [ ] El workflow de GitHub Actions pasó correctamente.
- [ ] Se actualizó documentación si corresponde.

---

### Ejemplo de PR de hotfix ya redactado (adaptar con su caso real):

**Título:** hotfix: corregir validación de token expirado

**Descripción:**
Se detectó que el endpoint de autenticación no rechazaba correctamente tokens expirados, permitiendo acceso indebido. Se corrigió la validación de expiración en el middleware de autenticación.

**Rama:** `hotfix/token-expirado` → `main`

**Cómo probarlo:** Generar un token con fecha de expiración pasada y verificar que la API responda 401 Unauthorized.
