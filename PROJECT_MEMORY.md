# Contexto de trabajo de EmailPro

Registrado por petición del usuario el 14 de septiembre de 2026.

- Verito prepara las interfaces visuales en `C:/wamp64/www/svr/EmailProVerito/`.
- Emi desarrolla las APIs.
- Roberto integra las interfaces de Verito con las APIs de Emi en `C:/wamp64/www/svr/SVR_EmailPro_WEB/`.
- `C:/wamp64/www/svr/EmailPro_PaginaWeb_Basica_Implementación/` es la plantilla pública sin inicio de sesión donde se reflejan las configuraciones del administrador. Durante las pruebas carga el sitio desde `/website/:domainId`.
- El usuario quiere aprender la integración paso a paso y capacitar a Roberto para realizarla también sin IA, reduciendo el consumo de tokens mediante ejemplos pequeños y reutilizables.
- Para el gestor web, conservar el diseño de Verito. Primera integración: creación del sitio al confirmar; las siguientes activaciones o desactivaciones no deben crear otro sitio.
- Avanzar por API y por CRUD. Consultar dudas antes de decidir comportamientos o contratos no especificados.

Esta es una memoria local del proyecto, no una memoria global de la cuenta. La revisión técnica y los pendientes están en `PLAN_INTEGRACION_SITIOS.md`.
