// CALENDARIO IMCA · Función de Supabase (Edge Function)
// Conecta el Gmail de cada terapeuta y crea citas en su Google Calendar.
// Llaves que usa (ya guardadas en Secrets): GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET.
// Las demás (SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, SUPABASE_ANON_KEY) las pone Supabase sola.

import { createClient } from "npm:@supabase/supabase-js@2";

const SUPABASE_URL = Deno.env.get("SUPABASE_URL")!;
const SERVICE_KEY = Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!;
const GOOGLE_ID = Deno.env.get("GOOGLE_CLIENT_ID")!;
const GOOGLE_SECRET = Deno.env.get("GOOGLE_CLIENT_SECRET")!;

const FUNCTION_BASE = `${SUPABASE_URL}/functions/v1/calendario`;
const APP_URL = "https://plataforma-imca.netlify.app";
const SCOPE = "https://www.googleapis.com/auth/calendar.events openid email";

const admin = createClient(SUPABASE_URL, SERVICE_KEY, { auth: { persistSession: false } });

const CORS = {
  "Access-Control-Allow-Origin": APP_URL,
  "Access-Control-Allow-Headers": "authorization, content-type, apikey, x-client-info",
  "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
};

function json(body: unknown, status = 200) {
  return new Response(JSON.stringify(body), {
    status,
    headers: { ...CORS, "content-type": "application/json" },
  });
}

function tokenDe(req: Request): string {
  return (req.headers.get("authorization") ?? "").replace("Bearer ", "").trim();
}

// Devuelve el id del usuario si el token es válido; si no, null.
async function usuarioDesdeToken(token: string): Promise<string | null> {
  if (!token) return null;
  const { data, error } = await admin.auth.getUser(token);
  if (error || !data.user) return null;
  return data.user.id;
}

async function rolDe(userId: string): Promise<string | null> {
  const { data } = await admin.from("perfiles").select("rol, activo").eq("id", userId).single();
  if (!data || !data.activo) return null;
  return data.rol;
}

// Firma para que nadie pueda inventar el "state" de Google y ligar su Gmail a otra persona.
async function firmar(texto: string): Promise<string> {
  const key = await crypto.subtle.importKey(
    "raw",
    new TextEncoder().encode(SERVICE_KEY),
    { name: "HMAC", hash: "SHA-256" },
    false,
    ["sign"],
  );
  const sig = await crypto.subtle.sign("HMAC", key, new TextEncoder().encode(texto));
  return [...new Uint8Array(sig)].map((b) => b.toString(16).padStart(2, "0")).join("");
}

// Puede cambiar el evento de un terapeuta: agenda/admin, o el propio terapeuta.
async function puedeTocarCalendario(userId: string, terapeutaId: string): Promise<boolean> {
  if (userId === terapeutaId) return true;
  const rol = await rolDe(userId);
  return rol === "admin" || rol === "agenda";
}

// Pide a Google un token de acceso nuevo usando el refresh token guardado.
async function accessTokenDe(perfilId: string): Promise<string> {
  const { data } = await admin
    .from("google_calendario")
    .select("refresh_token")
    .eq("perfil_id", perfilId)
    .maybeSingle();
  if (!data) throw new Error("Esta persona todavía no conecta su Google Calendar.");

  const r = await fetch("https://oauth2.googleapis.com/token", {
    method: "POST",
    headers: { "content-type": "application/x-www-form-urlencoded" },
    body: new URLSearchParams({
      client_id: GOOGLE_ID,
      client_secret: GOOGLE_SECRET,
      refresh_token: data.refresh_token,
      grant_type: "refresh_token",
    }),
  });
  const j = await r.json();
  if (!r.ok) throw new Error("Google no aceptó la conexión. Pide que vuelva a conectar su Gmail.");
  return j.access_token;
}

Deno.serve(async (req) => {
  if (req.method === "OPTIONS") return new Response(null, { headers: CORS });

  const url = new URL(req.url);
  const ruta = url.pathname.replace(/^.*\/calendario/, "") || "/";

  try {
    // 1) La persona inicia la conexión desde la plataforma
    if (ruta === "/conectar" && req.method === "GET") {
      const userId = await usuarioDesdeToken(url.searchParams.get("token") ?? "");
      if (!userId) return new Response("Sesión no válida. Vuelve a entrar a la plataforma.", { status: 401 });

      const g = new URL("https://accounts.google.com/o/oauth2/v2/auth");
      g.searchParams.set("client_id", GOOGLE_ID);
      g.searchParams.set("redirect_uri", `${FUNCTION_BASE}/callback`);
      g.searchParams.set("response_type", "code");
      g.searchParams.set("scope", SCOPE);
      g.searchParams.set("access_type", "offline");
      g.searchParams.set("prompt", "consent select_account");
      g.searchParams.set("state", `${userId}.${await firmar(userId)}`);
      return Response.redirect(g.toString(), 302);
    }

    // 2) Google regresa con el permiso y guardamos el refresh token
    if (ruta === "/callback" && req.method === "GET") {
      const code = url.searchParams.get("code");
      const [userId, sig] = (url.searchParams.get("state") ?? "").split(".");
      if (!code || !userId || !sig || (await firmar(userId)) !== sig) {
        return new Response("Conexión no válida.", { status: 400 });
      }

      const t = await fetch("https://oauth2.googleapis.com/token", {
        method: "POST",
        headers: { "content-type": "application/x-www-form-urlencoded" },
        body: new URLSearchParams({
          code,
          client_id: GOOGLE_ID,
          client_secret: GOOGLE_SECRET,
          redirect_uri: `${FUNCTION_BASE}/callback`,
          grant_type: "authorization_code",
        }),
      });
      const tj = await t.json();
      if (!t.ok || !tj.refresh_token) {
        const motivo = tj.error_description ?? tj.error ?? "sin detalle de Google";
        return new Response("Google no entregó permiso: " + motivo, { status: 400 });
      }

      const u = await fetch("https://openidconnect.googleapis.com/v1/userinfo", {
        headers: { authorization: `Bearer ${tj.access_token}` },
      });
      const uj = await u.json();

      const { error } = await admin.from("google_calendario").upsert({
        perfil_id: userId,
        correo_google: uj.email ?? "",
        refresh_token: tj.refresh_token,
        conectado: new Date().toISOString(),
      });
      if (error) return new Response("No se pudo guardar la conexión.", { status: 500 });

      return Response.redirect(`${APP_URL}/?calendario=ok`, 302);
    }

    // 3) La plataforma pregunta si la persona ya está conectada
    if (ruta === "/estado" && req.method === "GET") {
      const userId = await usuarioDesdeToken(tokenDe(req));
      if (!userId) return json({ error: "Sesión no válida." }, 401);
      const { data } = await admin
        .from("google_calendario")
        .select("correo_google")
        .eq("perfil_id", userId)
        .maybeSingle();
      return json({ conectado: !!data, correo: data?.correo_google ?? null });
    }

    // 4) Crear la cita en el Google Calendar del terapeuta (solo admin o agenda)
    if (ruta === "/crear-evento" && req.method === "POST") {
      const userId = await usuarioDesdeToken(tokenDe(req));
      if (!userId) return json({ error: "Sesión no válida." }, 401);
      const rol = await rolDe(userId);
      if (rol !== "admin" && rol !== "agenda") return json({ error: "Tu rol no puede agendar citas." }, 403);

      const b = await req.json();
      if (!b.terapeuta_id || !b.inicio || !b.fin || !b.titulo) {
        return json({ error: "Faltan datos de la cita." }, 400);
      }

      const access = await accessTokenDe(b.terapeuta_id);
      const r = await fetch("https://www.googleapis.com/calendar/v3/calendars/primary/events", {
        method: "POST",
        headers: { authorization: `Bearer ${access}`, "content-type": "application/json" },
        body: JSON.stringify({
          summary: b.titulo,
          description: b.descripcion ?? "",
          location: b.ubicacion ?? "",
          start: { dateTime: b.inicio, timeZone: "America/Cancun" },
          end: { dateTime: b.fin, timeZone: "America/Cancun" },
        }),
      });
      const ev = await r.json();
      if (!r.ok) return json({ error: "Google no pudo crear el evento." }, 502);
      return json({ id: ev.id, enlace: ev.htmlLink });
    }

    // 5) Cambiar título y notas de un evento (confirmación, asistencia, notas)
    if (ruta === "/actualizar-evento" && req.method === "POST") {
      const userId = await usuarioDesdeToken(tokenDe(req));
      if (!userId) return json({ error: "Sesión no válida." }, 401);
      const b = await req.json();
      if (!b.terapeuta_id || !b.evento_id) return json({ error: "Faltan datos del evento." }, 400);
      if (!(await puedeTocarCalendario(userId, b.terapeuta_id))) return json({ error: "No puedes cambiar este evento." }, 403);

      const access = await accessTokenDe(b.terapeuta_id);
      const r = await fetch(`https://www.googleapis.com/calendar/v3/calendars/primary/events/${encodeURIComponent(b.evento_id)}`, {
        method: "PATCH",
        headers: { authorization: `Bearer ${access}`, "content-type": "application/json" },
        body: JSON.stringify({ summary: b.titulo, description: b.descripcion ?? "" }),
      });
      if (!r.ok) return json({ error: "Google no pudo actualizar el evento." }, 502);
      return json({ ok: true });
    }

    // 6) Quitar un evento del calendario (cancelación o reprogramación)
    if (ruta === "/borrar-evento" && req.method === "POST") {
      const userId = await usuarioDesdeToken(tokenDe(req));
      if (!userId) return json({ error: "Sesión no válida." }, 401);
      const b = await req.json();
      if (!b.terapeuta_id || !b.evento_id) return json({ error: "Faltan datos del evento." }, 400);
      if (!(await puedeTocarCalendario(userId, b.terapeuta_id))) return json({ error: "No puedes cambiar este evento." }, 403);

      const access = await accessTokenDe(b.terapeuta_id);
      const r = await fetch(`https://www.googleapis.com/calendar/v3/calendars/primary/events/${encodeURIComponent(b.evento_id)}`, {
        method: "DELETE",
        headers: { authorization: `Bearer ${access}` },
      });
      // 404 o 410 quiere decir que el evento ya no existe: no es un error.
      if (!r.ok && r.status !== 404 && r.status !== 410) return json({ error: "Google no pudo borrar el evento." }, 502);
      return json({ ok: true });
    }

    return json({ error: "Ruta no encontrada." }, 404);
  } catch (e) {
    return json({ error: (e as Error).message ?? "Error inesperado." }, 500);
  }
});
