# Init Store VTEX IO — AI Skill

Skill para agentes AI que inicializa una tienda VTEX IO completa basada en Fast Template LATAM.

## Instalacion

```bash
infracode skills add github:InfracommerJuanPanchana/skill-init-store-io@init-store-io
```

Esto instala automaticamente en `.agents/`, `.claude/`, `.kilocode/`, `.opencode/` segun tu proyecto.

## Uso

```
/init-store-io
```

O decir: "Quiero iniciar una tienda VTEX IO"

## Las 3 Fases

| Fase | Datos | Resultado |
|------|-------|-----------|
| **1. Levantar** | VENDOR, PROYECTO, tipo tienda, pais | Tienda funcional linkeada en workspace `fasttemplate` |
| **2. Estilos** | Marca, colores, fuentes, imagenes | Branding aplicado, mails subidos |
| **3. Customs** | Integraciones, envio gratis | Tienda lista para QA |

## Requisitos

- Node.js >= 14
- Yarn global
- VTEX CLI
- Acceso SSH a Bitbucket (vtex-infralatam)
- Acceso al account VTEX

## Licencia

Uso interno — Infracommerce LATAM.
