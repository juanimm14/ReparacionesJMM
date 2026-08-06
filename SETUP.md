# Reparaciones JMM — Guía de configuración de la versión mejorada

Esta carpeta tiene:
- `index.html` → la versión nueva y mejorada (subir esta a GitHub Pages).
- `index.backup-2026-08-06.html` → copia exacta de tu archivo original, por si querés volver atrás.

## Qué cambió

1. **Login real con Firebase Authentication** (antes la contraseña `JMM2025` estaba escrita en el código, visible para cualquiera que abriera "Ver código fuente").
2. **Datos en tiempo real**: reparaciones, clientes y gastos se actualizan solos en todos los dispositivos abiertos, sin necesidad de recargar.
3. **Pestaña Dashboard** con tarjetas de resumen y 2 gráficos: ingresos vs. gastos por mes, y reparaciones por estado.
4. **Tabla de reparaciones**: ahora se puede ordenar clickeando cualquier encabezado de columna, y pagina de a 20 filas.

## Pasos que tenés que hacer vos en Firebase (una sola vez)

Sin estos 3 pasos el login no va a funcionar. Andá a [console.firebase.google.com](https://console.firebase.google.com) → proyecto **reparacionesjmm**.

### 1) Habilitar el método de acceso
`Authentication` → pestaña `Sign-in method` → `Add new provider` → elegí **Email/Password** → activalo → Guardar.

### 2) Crear tu usuario
`Authentication` → pestaña `Users` → `Add user` → cargá el email y la contraseña con la que vas a entrar de ahora en más (podés usar el email que quieras, no hace falta que sea real, pero recomendado usar uno tuyo de verdad).

Ese email/contraseña reemplaza a la contraseña vieja `JMM2025`.

### 3) Reglas de seguridad de Firestore (el paso más importante)

Hoy, aunque el login pida contraseña, cualquiera que sepa el `firebaseConfig` (que está a la vista en el código fuente, siempre pasa con Firebase) puede leer y escribir tu base de datos directamente si las reglas están abiertas. El login de la web NO alcanza para proteger los datos — hay que exigir autenticación también del lado de Firestore.

Andá a `Firestore Database` → pestaña `Rules` y reemplazá todo por esto:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Esto dice: "solo alguien que inició sesión (con el usuario que creaste en el paso 2) puede leer o escribir, nadie más". Después de pegarlo, clickeá **Publicar**.

### 4) Subir el `index.html` nuevo

Reemplazá el `index.html` de tu repo `ReparacionesJMM` en GitHub por el de esta carpeta (podés arrastrarlo directo en la web de GitHub, en "Add file → Upload files", o hacer commit desde tu editor). A los pocos segundos se actualiza solo en `https://juanimm14.github.io/ReparacionesJMM/`.

## Cómo probarlo antes de subirlo

Abrí `index.html` de esta carpeta haciendo doble clic (funciona local igual, porque ya apunta a tu proyecto de Firebase real). Si ves "Usuario o contraseña incorrectos" es porque todavía no hiciste el paso 2. Si ves error de permisos, es el paso 3.

## Si algo sale mal

Restaurá `index.backup-2026-08-06.html` (renombralo a `index.html` y volvé a subirlo) — ese archivo no tiene ninguno de estos cambios y sigue funcionando con la contraseña vieja, siempre que no hayas cambiado ya las reglas de Firestore del paso 3.
