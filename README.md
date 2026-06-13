# 🤝 Voluntariado — Plataforma de gestión de voluntariado comunitario

Plataforma web para que organizaciones publiquen eventos de voluntariado y que cualquier persona se inscriba **sin necesidad de crear cuenta**. Los coordinadores gestionan eventos e inscritos con un login seguro.

Construida con **Cloudflare Workers + Firebase Firestore**. Costo de operación: **$0** en planes gratuitos para uso comunitario.

---

## ✨ Características

- **Inscripción sin registro** — los voluntarios solo llenan un formulario
- **Campos configurables por evento** — el coordinador elige qué datos pedir (nombre, correo, celular, edad, matrícula, preguntas libres, etc.)
- **Privacidad por diseño** — el público solo ve nombres de pila y conteos; los datos sensibles (correos, teléfonos) solo los ven los coordinadores autenticados
- **Roles** — un administrador y varios coordinadores con permisos diferenciados
- **Estados flexibles** — eventos y actividades se pueden activar, desactivar, marcar como llenos u ocultar
- **Login sin contraseña** — Google o link mágico por correo
- **Seguridad robusta** — la base de datos está completamente cerrada; solo el Worker accede a ella

---

## 🏛 Arquitectura

```
┌──────────────┐      ┌──────────────────┐      ┌──────────────┐
│   Navegador  │─────▶│ Cloudflare Worker│─────▶│  Firestore   │
│  (frontend)  │◀─────│   (backend/API)  │◀─────│ (base datos) │
└──────────────┘      └──────────────────┘      └──────────────┘
      │                                                  ▲
      │            Firebase Auth (solo login)            │
      └──────────────────────────────────────────────────┘
```

**Principio clave:** el navegador nunca toca la base de datos directamente. Todo pasa por el Worker, que filtra qué información entrega según quién pregunta. Firestore está cerrado con `allow read, write: if false` — solo el Worker, con su llave de servicio, puede acceder.

---

## 🚀 Implementación

Hay dos rutas. Elige según tu nivel técnico.

### Ruta A — Desde el navegador (recomendada, sin instalar nada)

Para quien quiere su propia copia sin usar la terminal.

#### 1. Crea tu proyecto Firebase
1. Ve a [console.firebase.google.com](https://console.firebase.google.com) → **Agregar proyecto**
2. Crea una base de datos **Firestore** (modo producción)
3. En **Authentication → Sign-in method**, habilita:
   - **Google**
   - **Correo electrónico/Contraseña** + **Vínculo de correo (sin contraseña)**

#### 2. Genera la llave de servicio
1. **⚙️ Configuración → Cuentas de servicio**
2. **Generar nueva clave privada** → se descarga un `.json`
3. Guárdalo bien (lo necesitarás en el paso 4)

#### 3. Configura las reglas de Firestore
En **Firestore → Reglas**, pega esto y publica:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```
Sí, todo cerrado. El Worker accede con la llave de servicio, que ignora estas reglas.

#### 4. Despliega el Worker en Cloudflare
1. Haz **fork** de este repositorio en GitHub
2. En [dash.cloudflare.com](https://dash.cloudflare.com) → **Workers & Pages → Create → Pages → Connect to Git**
3. Selecciona tu fork
4. En **Settings → Environment Variables**, agrega:

| Variable | Valor (del archivo .json) |
|---|---|
| `FIREBASE_PROJECT_ID` | campo `project_id` |
| `FIREBASE_CLIENT_EMAIL` | campo `client_email` |
| `FIREBASE_PRIVATE_KEY` | campo `private_key` (completo, con los `\n`) |
| `ADMIN_EMAIL` | tu correo de administrador |

5. Despliega

#### 5. Conecta el frontend con tu Firebase
En `public/index.html`, busca `firebaseConfig` y reemplaza con los datos de **tu** proyecto (Configuración → Tus apps → Web).

#### 6. Autoriza tu dominio
En Firebase → **Authentication → Settings → Dominios autorizados**, agrega la URL que te dio Cloudflare (algo como `tu-proyecto.pages.dev`).

¡Listo! 🎉

---

### Ruta B — Con Wrangler (para desarrolladores)

Requiere [Node.js](https://nodejs.org) instalado.

```bash
# 1. Clona el repo
git clone https://github.com/TU_USUARIO/voluntariado.git
cd voluntariado

# 2. Instala dependencias
npm install

# 3. Configura los secrets locales
cp .dev.vars.example .dev.vars
# edita .dev.vars con los datos de tu service account

# 4. Prueba localmente
npm run dev

# 5. Sube los secrets a Cloudflare
wrangler secret put FIREBASE_PROJECT_ID
wrangler secret put FIREBASE_CLIENT_EMAIL
wrangler secret put FIREBASE_PRIVATE_KEY
wrangler secret put ADMIN_EMAIL

# 6. Despliega
npm run deploy
```

---

## 📂 Estructura del proyecto

```
voluntariado/
├── src/
│   └── worker.js          Backend: API REST que filtra datos y verifica permisos
├── public/
│   └── index.html         Frontend: la interfaz de usuario
├── wrangler.toml          Configuración de Cloudflare Workers
├── package.json           Dependencias del proyecto
├── .gitignore             Protege archivos sensibles
├── .dev.vars.example      Plantilla de configuración local
└── README.md              Este archivo
```

---

## 🔐 Seguridad

- **Base de datos cerrada**: Firestore rechaza todo acceso directo (`if false`). Solo el Worker entra, usando una cuenta de servicio.
- **Datos sensibles filtrados**: el endpoint público (`/api/eventos`) solo devuelve nombres de pila y conteos. Los correos y teléfonos solo se entregan en endpoints protegidos que verifican el token de Firebase Auth del coordinador.
- **Secrets fuera del código**: la llave privada nunca está en el repositorio; vive como variable de entorno cifrada en Cloudflare.
- **Validación del lado del servidor**: el Worker valida cupos, duplicados y permisos. No confía en el cliente.

---

## 📝 Licencia

MIT — úsalo, modifícalo y compártelo libremente.

---

## 🙏 Créditos

Desarrollado para la iniciativa Distrito Tec. Inspirado en la necesidad de herramientas comunitarias accesibles y sin costo.
