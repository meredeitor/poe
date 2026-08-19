# Caborca POE

PWA para crear, publicar y consultar Procedimientos Operacionales Estándar. Firestore conserva el contenido y las fotografías optimizadas; PDF y QR se generan bajo demanda. El proyecto no utiliza Cloud Storage y está diseñado para operar en Firebase Spark.

## Configuración Firebase

La aplicación está conectada al proyecto Firebase `leantakt-7beac` mediante el SDK web 12.17.1. La propiedad `storageBucket` no se utiliza porque las imágenes se guardan optimizadas en Firestore.

1. Crea un proyecto en [Firebase Console](https://console.firebase.google.com/).
2. En **Authentication → Sign-in method**, activa **Correo electrónico/contraseña** y crea el primer usuario.
3. Crea Firestore en modo producción. No habilites Cloud Storage ni vincules una cuenta de facturación.
4. En **Configuración del proyecto → Tus apps**, registra una app web y copia sus valores a `js/firebase-config.js`.
5. Crea manualmente `users/{UID}` en Firestore con: `role: "Administrador"`, `active: true`, `email: "..."`.
6. Instala Firebase CLI, inicia sesión y vincula el proyecto: `firebase login`, `firebase use --add`.
7. Publica reglas e índices: `firebase deploy --only firestore:rules,firestore:indexes`.
8. Publica el sitio: `firebase deploy --only hosting`.

No publiques credenciales privadas ni claves de cuentas de servicio. La configuración web de Firebase identifica la app, pero la autorización real depende de Authentication y las Rules.

## Modelo de datos

```text
areas / procesos / maquinas / puestos
users/{uid}
poes/{poeId}
  revisions/{revisionId}
    steps/{stepId}
      images/main
auditLog/{eventId}
consultas/{eventId}
```

El documento POE conserva `currentRevisionId`; crear una revisión solo establece `draftRevisionId`. Al publicar, una transacción vuelve obsoleta la anterior, publica la nueva y actualiza `currentRevisionId`. El QR apunta siempre a `poe.html?id=CODIGO`, nunca a una revisión.

Cada fotografía se convierte en WebP dentro del navegador, se limita a 1280 px y se reduce a un máximo aproximado de 700 KB antes de guardarse en `images/main`. Nunca se incluye la imagen en el documento principal del paso, evitando acercarlo al límite de 1 MiB de Firestore. Una revisión nueva copia tanto los pasos como sus documentos de imagen.

## Ejecución local

Sirve la carpeta mediante HTTP (los módulos ES y el Service Worker no funcionan correctamente abriendo los HTML como archivos). Ejecuta `npm start` o cualquier servidor estático.

## Publicación en GitHub Pages

1. Crea un repositorio de GitHub y coloca el contenido de `Caborca_POE` en su raíz.
2. Sube los archivos a la rama `main`. El flujo `.github/workflows/pages.yml` publicará el sitio automáticamente.
3. En GitHub abre **Settings → Pages** y selecciona **GitHub Actions** como origen.
4. Copia el dominio resultante, por ejemplo `usuario.github.io`, y agrégalo en Firebase Console → Authentication → Settings → Authorized domains.
5. Publica por separado `firestore.rules` y `firestore.indexes.json` mediante Firebase CLI. GitHub Pages publica la interfaz, pero no configura Firestore.

Todas las rutas de la PWA son relativas, por lo que funcionan tanto en `usuario.github.io/repositorio/` como en un dominio personalizado. El QR se genera con la ubicación publicada actual e incluye automáticamente el nombre del repositorio.

## Alcance de esta versión

Incluye búsqueda pública, visor paso a paso, panel responsive, catálogos, creación de POE, pasos con imágenes WebP guardadas en Firestore, borradores, revisiones, publicación transaccional, auditoría, QR descargable/imprimible, PDF bajo demanda, caché offline, reglas e índices. La asignación segura de roles debe hacerse desde Firebase Console o desde un backend privilegiado; nunca desde el navegador.

## Editar, eliminar y retirar procedimientos

- **Borrador:** usa `Editar` para modificar información, pasos o fotografías; `Eliminar` borra definitivamente el borrador.
- **En revisión:** usa `Editar` para continuar trabajando; `Descartar` elimina únicamente la revisión en preparación y conserva la revisión vigente.
- **Vigente:** crea primero una `Nueva revisión`. La revisión publicada permanece disponible hasta publicar la nueva.
- **Obsoletar o cancelar:** retira el POE de la consulta pública y del QR, pero conserva todo su historial documental.
- Los POE publicados nunca se eliminan físicamente desde la interfaz.

## Permanecer en el plan gratuito

- Mantén el proyecto en Spark y no vincules Cloud Billing.
- Revisa periódicamente **Firestore → Uso**: el límite gratuito es 1 GiB almacenado, 50,000 lecturas diarias, 20,000 escrituras diarias y 10 GiB mensuales de salida.
- Las imágenes son el consumo principal. El navegador las comprime antes de guardarlas y las reglas rechazan documentos superiores al límite definido.
- No se admiten videos en Firestore. Utiliza únicamente enlaces externos si esa función se habilita después.
- Si se alcanza una cuota Spark, Firebase suspende temporalmente ese producto en lugar de generar un cobro.
