Lane (swift programming fan) Seals - guitar playing - EDUCATIONAL USE and  Educational Purposes ONLY - * 

print("test")
#python ANDOR swift is cool

[www.guitarmapfree.com is lit!]

---------------------------------

List of O.P. music videos Artists :

["Tate, Billy, AdiR, L.A.x., bb, sh, ++]

-----------------------------------

#code
calicocoin beta edu release

console.log("test");

import express from "express";
import cors from "cors";
import morgan from "morgan";
import jwt from "jsonwebtoken";

const app = express();
app.use(cors());
app.use(express.json());
app.use(morgan("dev"));

const SECRET = process.env.TOGGLE_API_SECRET || "dev-super-secret";
const ISSUER = "toggle-token-api";
const toggles = new Map(); // { name: boolean }

// --- helpers ---
function signToken({ scope = ["toggle:*"], ttl = 3600, sub = "client" } = {}) {
  const now = Math.floor(Date.now() / 1000);
  const payload = { iss: ISSUER, iat: now, scope, sub };
  return jwt.sign(payload, SECRET, { expiresIn: ttl });
}
function authMiddleware(requiredScopePrefix = "toggle:") {
  return (req, res, next) => {
    const auth = req.headers.authorization || "";
    const token = auth.startsWith("Bearer ") ? auth.slice(7) : null;
    if (!token) return res.status(401).json({ error: "Missing bearer token" });
    try {
      const decoded = jwt.verify(token, SECRET);
      req.token = decoded;
      // simple scope check
      const ok = (decoded.scope || []).some(s => s === `${requiredScopePrefix}*` || s.startsWith(requiredScopePrefix));
      if (!ok) return res.status(403).json({ error: "Insufficient scope" });
      next();
    } catch (e) {
      return res.status(401).json({ error: "Invalid/expired token" });
    }
  };
}

// --- routes ---
app.get("/", (_req, res) => {
  res.json({ name: "Toggle Token API", issuer: ISSUER, endpoints: ["/api/token", "/api/toggle/:name", "/api/toggles"] });
});

// Mint a token
app.post("/api/token", (req, res) => {
  const { scope, ttl, sub } = req.body || {};
  const token = signToken({ scope: scope || ["toggle:*"], ttl: ttl || 3600, sub: sub || "client" });
  res.json({ token, expires_in: ttl || 3600, scope: scope || ["toggle:*"] });
});

// Get a toggle
app.get("/api/toggle/:name", authMiddleware("toggle:"), (req, res) => {
  const name = req.params.name;
  const value = toggles.get(name) ?? false;
  res.json({ name, value });
});

// Flip or set a toggle
// body: { set?: boolean }  (if omitted, it flips)
app.post("/api/toggle/:name", authMiddleware("toggle:"), (req, res) => {
  const name = req.params.name;
  const { set } = req.body || {};
  const current = toggles.get(name) ?? false;
  const next = typeof set === "boolean" ? set : !current;
  toggles.set(name, next);
  res.json({ name, previous: current, value: next });
});

// List all toggles
app.get("/api/toggles", authMiddleware("toggle:"), (_req, res) => {
  res.json(Object.fromEntries(toggles.entries()));
});

// Health
app.get("/healthz", (_req, res) => res.json({ ok: true }));

const port = process.env.PORT || 3000;
app.listen(port, () => console.log(`Toggle Token API running on http://localhost:${port}`));

