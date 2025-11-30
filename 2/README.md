
 #  **el HTML actualizado** 


con los scripts }
(fecha de primer *publish*/commit guardada, última edición, y funciones opcionales para obtener 
commits desde GitHub si quieres mostrar el primer/último commit real).

Además al final  una **base mínima tipo “Facebook”** (MVP para el primer año):
arquitectura, stack, modelo de datos y roadmap corto.

Copia-pega este HTML tal cual en tu hosting (Vercel, Netlify, o carpeta estática). TODO funciona sin backend si no tienes repo: usa `localStorage`. Si quieres mostrar commits reales de GitHub, sigue las instrucciones dentro del código (necesitas repo owner/name y opcional token).

---

## HTML + JS (publicación / commits / fechas)

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Lo Que No Vieron de Mí — Publicar</title>
  <style>
    body{font-family:Inter,Arial,Helvetica,sans-serif;background:#f6f7fb;color:#111;margin:0;padding:24px}
    .wrap{max-width:900px;margin:20px auto;background:#fff;padding:28px;border-radius:12px;box-shadow:0 6px 24px rgba(20,20,40,0.06)}
    h1{margin:0 0 12px}
    .meta{display:flex;gap:12px;flex-wrap:wrap;margin:16px 0}
    .chip{background:#f3f4f7;padding:10px 12px;border-radius:10px;font-size:14px;color:#333}
    .btn{background:#2563eb;color:#fff;padding:8px 12px;border-radius:8px;border:0;cursor:pointer}
    textarea{width:100%;min-height:160px;border-radius:8px;padding:12px;border:1px solid #e1e4ee;resize:vertical}
    .small{font-size:13px;color:#666}
    .row{display:flex;gap:8px;flex-wrap:wrap}
    a{color:#2563eb}
    footer{margin-top:20px;font-size:13px;color:#666}
  </style>
</head>
<body>
  <div class="wrap">
    <h1>Lo Que No Vieron de Mí</h1>

    <p class="small">Edita aquí tu texto y usa <strong>Publicar</strong> para establecer la fecha de primera subida. Cada vez que guardes se registrará la <em>última edición</em>. Si quieres, conecta GitHub para mostrar primer/último commit real.</p>

    <textarea id="content">(Pega o edita tu texto aquí)...</textarea>

    <div style="margin-top:12px" class="row">
      <button class="btn" id="publishBtn">Publicar (guardar fecha de primera subida)</button>
      <button class="btn" id="saveBtn" style="background:#10b981">Guardar (última edición)</button>
      <button class="btn" id="resetBtn" style="background:#ef4444">Reset fechas (local)</button>
    </div>

    <div class="meta" style="margin-top:18px">
      <div class="chip">Primera publicación: <strong id="firstPublished">—</strong></div>
      <div class="chip">Última edición: <strong id="lastEdited">—</strong></div>
      <div class="chip">Primer commit (GitHub): <strong id="ghFirst">—</strong></div>
      <div class="chip">Último commit (GitHub): <strong id="ghLast">—</strong></div>
    </div>

    <div style="margin-top:10px">
      <label class="small">Opcional: Mostrar commits reales desde GitHub (owner/repo). Si usas token, mantenlo privado.</label>
      <div style="display:flex;gap:8px;margin-top:8px;flex-wrap:wrap">
        <input id="ghOwner" placeholder="owner (ej. usuario)" style="padding:8px;border-radius:8px;border:1px solid #ddd" />
        <input id="ghRepo" placeholder="repo (ej. mi-proyecto)" style="padding:8px;border-radius:8px;border:1px solid #ddd" />
        <input id="ghToken" placeholder="token (opcional, mejor para privado)" style="padding:8px;border-radius:8px;border:1px solid #ddd" />
        <button class="btn" id="getGh">Traer commits GitHub</button>
      </div>
      <div style="margin-top:8px" class="small">Nota: la versión local usa <code>localStorage</code>. Si despliegas en servidor, cambia a DB (ver README abajo).</div>
    </div>

    <footer>
      Hecho por: Bio Intelligence One Universe Soul — Universe City
    </footer>
  </div>

<script>
/*
  Funcionalidad:
  - Guarda 'firstPublished' la primera vez que presionas Publicar en localStorage.
  - Cada vez que presionas Guardar guarda 'lastEdited' con timestamp actual.
  - Reset limpia localStorage local (solo pruebas).
  - Opcional: si proves owner/repo (y token), consulta GitHub API para obtener primer y último commit.
*/

const el = id => document.getElementById(id);
const ISO = d => (new Date(d)).toLocaleString(); // presentación local

function showLocalDates(){
  const first = localStorage.getItem('firstPublished');
  const last = localStorage.getItem('lastEdited');
  el('firstPublished').textContent = first ? ISO(first) : '—';
  el('lastEdited').textContent = last ? ISO(last) : '—';
}
showLocalDates();

el('publishBtn').addEventListener('click', () => {
  const content = el('content').value || '';
  // Si no existe firstPublished, lo guardamos
  if(!localStorage.getItem('firstPublished')){
    const now = new Date().toISOString();
    localStorage.setItem('firstPublished', now);
    // opcional: guardar snapshot del contenido
    localStorage.setItem('firstPublishedContent', content);
  }
  // siempre actualizar lastEdited porque publicas ahora
  const now2 = new Date().toISOString();
  localStorage.setItem('lastEdited', now2);
  localStorage.setItem('lastEditedContent', content);
  showLocalDates();
  alert('Publicado: fechas guardadas en local (para backend conectar con API si quieres).');
});

el('saveBtn').addEventListener('click', () => {
  const content = el('content').value || '';
  const now = new Date().toISOString();
  localStorage.setItem('lastEdited', now);
  localStorage.setItem('lastEditedContent', content);
  showLocalDates();
  alert('Guardado: última edición actualizada.');
});

el('resetBtn').addEventListener('click', () => {
  if(confirm('Resetear fechas locales? (esto solo borra localStorage del navegador)')) {
    localStorage.removeItem('firstPublished');
    localStorage.removeItem('firstPublishedContent');
    localStorage.removeItem('lastEdited');
    localStorage.removeItem('lastEditedContent');
    el('ghFirst').textContent = '—';
    el('ghLast').textContent = '—';
    showLocalDates();
  }
});

// --- GitHub optional ---
async function fetchGitHubCommits(owner, repo, token){
  if(!owner || !repo) throw new Error('owner/repo required');
  const headers = token ? { Authorization: 'token ' + token } : {};
  // Get commits (paginated). We'll ask first page and last page trick:
  // 1) fetch first page (most recent)
  const recentUrl = `https://api.github.com/repos/${owner}/${repo}/commits?per_page=1`;
  const firstPage = await fetch(recentUrl, { headers });
  if(!firstPage.ok) throw new Error('Cannot fetch commits (check owner/repo or rate limit)');
  const recent = await firstPage.json();
  const lastCommitSha = recent[0] ? recent[0].sha : null;
  // To get the earliest commit we need to get the commits list and read the last page.
  // GitHub uses Link header for pagination. We'll request a large per_page fallback if Link missing.
  const perPage = 100;
  const pageUrl = `https://api.github.com/repos/${owner}/${repo}/commits?per_page=${perPage}`;
  const r2 = await fetch(pageUrl, { headers });
  if(!r2.ok) throw new Error('No commits or access denied');
  const commitsPage = await r2.json();
  // If results < perPage -> earliest = last element of commitsPage
  let earliest = null;
  if(Array.isArray(commitsPage) && commitsPage.length > 0){
    if(commitsPage.length < perPage){
      earliest = commitsPage[commitsPage.length - 1];
    } else {
      // Fallback: try to follow Link header to get last page
      const link = r2.headers.get('Link');
      if(link){
        const m = link.match(/<([^>]+)>;\s*rel="last"/);
        if(m){
          const lastUrl = m[1];
          const lastResp = await fetch(lastUrl, { headers });
          const lastCommits = await lastResp.json();
          earliest = lastCommits[lastCommits.length - 1];
        }
      }
    }
  }
  return {
    last: recent[0] || null,
    first: earliest || null
  };
}

el('getGh').addEventListener('click', async () => {
  const owner = el('ghOwner').value.trim();
  const repo = el('ghRepo').value.trim();
  const token = el('ghToken').value.trim();
  el('ghFirst').textContent = 'cargando...';
  el('ghLast').textContent = 'cargando...';
  try{
    const {first, last} = await fetchGitHubCommits(owner, repo, token || undefined);
    if(first){
      el('ghFirst').textContent = `${first.sha.slice(0,7)} • ${new Date(first.commit.committer.date).toLocaleString()}`;
    } else {
      el('ghFirst').textContent = '—';
    }
    if(last){
      el('ghLast').textContent = `${last.sha.slice(0,7)} • ${new Date(last.commit.committer.date).toLocaleString()}`;
    } else {
      el('ghLast').textContent = '—';
    }
  } catch(err){
    el('ghFirst').textContent = 'error';
    el('ghLast').textContent = 'error';
    alert('Error GitHub: ' + err.message);
    showLocalDates();
  }
});

// Auto-load content from lastEditedContent if existe (UX)
const lastContent = localStorage.getItem('lastEditedContent') || localStorage.getItem('firstPublishedContent');
if(lastContent) el('content').value = lastContent;
</script>
</body>
</html>
```

---

## Cómo funciona y qué sigue

* **Sin GitHub**: todo queda en `localStorage` del navegador. `firstPublished` se asigna la primera vez que presionas **Publicar**; `lastEdited` se actualiza al guardar. Ideal para pruebas y para publicar rápido sin servidor.
* **Con GitHub**: pega `owner` y `repo`, opcional token (mejor si privado). El script intenta traer el commit más reciente y el más antiguo; si tu repo es grande, puede necesitar token por rate-limit o paginación extra.
* **Producción (recomendado)**: crea una API (Node/Express, Firebase, Supabase) y guarda `firstPublished`, `lastEdited` y snapshots del contenido en DB. Así se comparte entre dispositivos y no depende del navegador.

---

## Ahora: mini-guía para crear una **base tipo Facebook** (MVP primer año)

Te dejo la **hoja de ruta mínima y el stack** para que construyas algo funcional, simple y escalable.

### 1) Objetivo MVP (primer año)

* Registro / login (email, móvil, OAuth opcional).
* Perfil de usuario (nombre, bio, avatar).
* Post simple (texto + foto + timestamp).
* Feed cronológico (posts de usuarios seguidos).
* Likes + comentarios.
* Notificaciones básicas (nuevo comentario, like).
* Moderación mínima (report).
* API simple y cliente web/mobile.

### 2) Tech stack recomendado (rápido y barato)

* Frontend: React (Vite) + Tailwind (o tu HTML si prefieres simple).
* Backend: Node.js + Express o Fastify.
* DB: PostgreSQL (relacional) o Supabase (rápido para empezar).
* File storage: S3 / DigitalOcean Spaces / or Vercel storage.
* Auth: Firebase Auth / Supabase Auth / NextAuth.
* Hosting: Vercel/Netlify (frontend) + Render/Heroku/Cloud Run (backend) or Supabase (todo en uno).
* Escalado: separar DB y storage; cache con Redis.

### 3) Modelo de datos (simplificado)

* users: id, username, email, hashed_password, bio, avatar_url, created_at
* posts: id, user_id, content, media_url, created_at, edited_at
* follows: follower_id, followee_id, created_at
* likes: id, user_id, post_id, created_at
* comments: id, user_id, post_id, content, created_at
* notifications: id, user_id, type, payload, read, created_at

### 4) Endpoints esenciales (REST)

* POST /signup, POST /login
* GET /users/:id/profile
* POST /posts, GET /posts/:id, GET /feed?user=me
* POST /posts/:id/like, POST /posts/:id/comment
* POST /follow/:id
* GET /notifications

### 5) Seguridad & datos

* Hashear contraseñas (bcrypt).
* Validar input (sanitize).
* Rate limit para endpoints de post y comentarios.
* Política CORS.
* Moderación: flag/reporte y revisión manual.

### 6) Roadmap 12 meses (milestones)

* Mes 0-1: MVP backend + frontend minimal (registro, post, feed).
* Mes 2-3: Likes, comentarios, perfil, media upload.
* Mes 4-6: Notificaciones, seguir/seguir, búsqueda básica.
* Mes 7-9: Moderación, panel admin, métricas básicas.
* Mes 10-12: Mobile responsive, optimizaciones, backups, posible app React Native/Capacitor.

### 7) UX simple pero efectivo

* Feed claro, desacoplado (server-side pagination).
* Subir media con compresión en cliente.
* Mostrar timestamps humanos (hace 3h).
* Usar websockets / Supabase Realtime para notificaciones en vivo cuando escales.

---

## Consejos propios (desde tu visión Universe City / Bio US)

* Empieza con lo mínimo y documenta TODO en tus páginas (eso demuestra empresa real).
* Cada vez que publiques algo importante, el script que te di ya guarda fechas. Luego migra eso a DB con endpoint `/publish` que registre autor y hashes de contenido (prueba de existencia).
* Si quieres replicar WhatsApp ligero: usa WebSocket para mensajes, cifrado simple en tránsito (TLS), y DB con mensajes por canal. Pero para red social, primero feed y perfiles.

---

¿Ahora qué quieres que haga yo ya mismo?

* Puedo adaptar el HTML para que suba el contenido a Supabase (te doy el código listo y cómo configurarlo).
* O lo preparo para que al presionar *Publicar* haga un commit automático a un repo (necesita token server-side).
* O te doy el esquema de DB en SQL listo para pegar.
