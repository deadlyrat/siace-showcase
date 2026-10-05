<div align="center">

<img src="assets/banner.gif" width="100%" alt="Banner animado de SIACE, Sistema Académico Escolar">

# SIACE

![Privado](https://img.shields.io/badge/C%C3%B3digo-Privado%20%C2%B7%20Proyecto%20Cliente-red?style=flat)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
![NestJS](https://img.shields.io/badge/NestJS%2010-E0234E?style=flat&logo=nestjs&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js%2015-000000?style=flat&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React%2019-61DAFB?style=flat&logo=react&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL%2016-4169E1?style=flat&logo=postgresql&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma%205-2D3748?style=flat&logo=prisma&logoColor=white)
![Redis](https://img.shields.io/badge/Redis%207-DC382D?style=flat&logo=redis&logoColor=white)
![Keycloak](https://img.shields.io/badge/Keycloak%2026-4D4D4D?style=flat&logo=keycloak&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)

**Sistema académico escolar para el contexto de MEDUCA (Panamá): matrícula, notas por trimestre, asistencia y boletines en PDF verificables, con acceso por roles y consulta de elegibilidad para IFARHU.**

</div>

> Este es un **portafolio showcase**: el código fuente es propietario y no está incluido.

---

## Contenido

- [El Problema](#el-problema)
- [La Solución](#la-solución)
- [Funcionalidades](#funcionalidades)
- [Vista Previa](#vista-previa)
- [Arquitectura](#arquitectura)
- [Stack Tecnológico](#stack-tecnológico)
- [Instalación local](#instalación-local)
- [Roadmap](#roadmap)
- [Contacto](#contacto)

---

## El Problema

Los centros educativos necesitan llevar sus registros académicos de forma ordenada y confiable:

- Registrar matrícula, notas y asistencia sin hojas de cálculo dispersas.
- Calcular la nota final ponderada por período y saber quién promueve.
- Entregar boletines que se puedan verificar y no se falsifiquen fácilmente.
- Dar a cada persona solo el acceso que le corresponde (docente, secretaría, dirección, ministerio).
- Importar los datos de un sistema anterior sin empezar de cero.

---

## La Solución

Un monorepo con una API REST en NestJS y una aplicación web en Next.js. Centraliza escuelas, años académicos con tres períodos, secciones, matrícula, notas y asistencia, y genera boletines en PDF con un hash de verificación. La autenticación se delega en Keycloak (OIDC) y cada pantalla y endpoint se restringe por rol. Incluye scripts de extracción y carga para importar datos de un sistema legado y una API de elegibilidad protegida con clave para IFARHU.

---

## Funcionalidades

| Funcionalidad | Descripción |
|---------------|-------------|
| Roles y permisos | Cinco roles (`DOCENTE`, `SECRETARIA`, `DIRECTOR`, `ADMIN_MEDUCA`, `IFARHU`) con guardas por endpoint y navegación filtrada por rol |
| Años académicos y períodos | Tres períodos por año, pesos por período y umbral de promoción configurables, con estado de ingreso de notas por período |
| Matrícula y secciones | Alta de grupos por escuela, ofertas de curso y matrícula de estudiantes con estados (activa, retirada, trasladada, graduada) |
| Ingreso de notas | Registro por oferta de curso, plantilla e importación desde Excel, con historial de revisiones por nota |
| Asistencia | Registro masivo (presente, ausente, tarde, excusado) y consulta por matrícula |
| Notas finales | Cálculo de la nota final ponderada por período y determinación de promoción según el umbral |
| Boletines en PDF | Generación individual o masiva en segundo plano (BullMQ y Puppeteer), almacenamiento en MinIO, aprobación y descarga |
| Verificación de boletines | Cada boletín lleva un hash SHA-256 y una URL de verificación |
| Elegibilidad IFARHU | Consulta por cédula con autenticación por clave de API |
| Administración y seguridad | Estadísticas, registros de auditoría, Helmet con CSP y HSTS, CORS restringido y límite de peticiones |

---

## Vista Previa

<table>
  <tr>
    <td width="50%">
      <img src="assets/cards/01-notas-y-boletines.png" width="100%" alt="Tarjeta sobre notas por trimestre y boletines en PDF">
      <br><b>Notas y boletines</b>: notas por período y boletines en PDF con hash de verificación.
    </td>
    <td width="50%">
      <img src="assets/cards/02-matricula-y-asistencia.png" width="100%" alt="Tarjeta sobre matrícula y asistencia">
      <br><b>Matrícula y asistencia</b>: estudiantes por sección y registro masivo de asistencia.
    </td>
  </tr>
  <tr>
    <td width="50%">
      <img src="assets/cards/03-acceso-seguro.png" width="100%" alt="Tarjeta sobre acceso seguro por roles">
      <br><b>Acceso seguro</b>: inicio de sesión con Keycloak y permisos por rol.
    </td>
    <td width="50%"></td>
  </tr>
</table>

---

## Arquitectura

```mermaid
graph LR
    USER["Navegador<br/>Next.js 15 · React 19<br/>NextAuth · TanStack Query"]
    API["API REST<br/>NestJS 10 · Swagger<br/>Guardas JWT y roles"]
    KC["Keycloak<br/>OIDC · JWKS"]
    PG[("PostgreSQL 16<br/>Prisma")]
    REDIS[("Redis 7<br/>Cola BullMQ")]
    WORKER["Procesador de boletines<br/>Puppeteer"]
    MINIO[("MinIO<br/>PDF de boletines")]
    IFARHU["Cliente IFARHU<br/>Clave de API"]

    USER -->|"Inicio de sesión"| KC
    USER -->|"Peticiones con token"| API
    API -->|"Validación del token"| KC
    API -->|"Lectura y escritura"| PG
    API -->|"Encola trabajos"| REDIS
    REDIS --> WORKER
    WORKER -->|"Lee datos"| PG
    WORKER -->|"Sube el PDF"| MINIO
    API -->|"Descarga del PDF"| MINIO
    IFARHU -->|"Consulta de elegibilidad"| API
```

**Módulos de la API:** autenticación, escuelas, usuarios, años académicos, cursos, grupos, estudiantes, matrículas, notas, asistencia, boletines, administración e IFARHU. Las rutas llevan el prefijo `/api` con versionado por URI y documentación Swagger.

**Despliegue:** composiciones de Docker para desarrollo, pruebas y producción, esta última con Nginx como proxy.

---

## Stack Tecnológico

| Capa | Tecnología |
|------|-----------|
| Monorepo | pnpm workspaces · Turborepo |
| Frontend | Next.js 15 · React 19 · TypeScript · Tailwind CSS |
| Interfaz | Radix UI · TanStack Query y Table · React Hook Form |
| Autenticación | Keycloak (OIDC) · NextAuth · Passport JWT |
| Backend | NestJS 10 · Swagger · Throttler · Helmet · class-validator · Zod |
| Datos | PostgreSQL 16 · Prisma 5 |
| Colas | Redis 7 · BullMQ |
| Archivos y PDF | MinIO · Puppeteer · xlsx |
| Pruebas | Jest |
| Infraestructura | Docker Compose · Nginx |
| Migración de datos | Scripts en Python |

---

## Instalación local

> **Aviso:** el código es privado y propietario. Estos pasos son solo para colaboradores autorizados con acceso al repositorio.

1. Instala Node.js 20 o superior, pnpm y Docker con el plugin `compose`.
2. Instala las dependencias del monorepo:
   ```bash
   pnpm install
   ```
3. Copia `.env.example` a `.env` y completa tus propios valores.
4. Levanta la infraestructura y los servicios (PostgreSQL, Redis, MinIO, Keycloak, API y web):
   ```bash
   docker compose up -d
   ```
5. Genera el cliente de Prisma, aplica las migraciones y carga los datos semilla:
   ```bash
   pnpm db:generate
   pnpm db:migrate
   pnpm db:seed
   ```
6. Para desarrollar sin contenedores de aplicación:
   ```bash
   pnpm dev
   ```

---

## Roadmap

- [ ] Integración continua con GitHub Actions.
- [ ] Autenticación de múltiples factores (MFA) en Keycloak.
- [ ] Políticas de seguridad por fila (RLS) en PostgreSQL.
- [ ] Pruebas de extremo a extremo con Playwright (inicio de sesión, notas y boletín).
- [ ] Prueba de carga con 50 sesiones concurrentes y cobertura de pruebas de al menos 80 %.

---

## Contacto

El código fuente es propietario. Para consultas o propuestas, escríbeme:

[![Email](https://img.shields.io/badge/Email-pablozam1931%40gmail.com-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:pablozam1931@gmail.com)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-(507)%206517--1870-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)](https://wa.me/50765171870)
[![GitHub](https://img.shields.io/badge/GitHub-deadlyrat-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/deadlyrat)

- Correo: [pablozam1931@gmail.com](mailto:pablozam1931@gmail.com)
- WhatsApp: [(507) 6517-1870](https://wa.me/50765171870)
- GitHub: [github.com/deadlyrat](https://github.com/deadlyrat)

---

*Parte del portafolio de [deadlyrat](https://github.com/deadlyrat)*
