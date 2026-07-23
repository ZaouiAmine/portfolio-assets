# portfolio-assets

Public static assets (images, video) for [devzaoui_portfolio](https://github.com/ZaouiAmine/tb_devzaoui_portfolio) Taubyte websites.

| Folder | Site path |
|--------|-----------|
| `fluxpath/` | `/fluxpath` |
| `velari/` | `/velari` |
| `tidalloom/` | `/tidalloom` |
| `verdifi/` | `/verdifi` |
| `reelpin/` | `/reelpin` |
| `portfolio/` | `/` (main portfolio) |

Raw URL pattern:

```
https://raw.githubusercontent.com/ZaouiAmine/portfolio-assets/main/<folder>/<file>
```
```
 component App() {
      let mut draft: String = "";
      let mut wall: String = "";
      let mut count: i32 = 0;
      fn set_draft(&mut self, value: String) {
          self.draft = value;
      }
      fn post(&mut self) {
          self.count = self.count + 1;
          self.wall = self.draft;
          self.draft = "";
      }
      render {
          <div class="wrap">
              <p class="eyebrow">"Dream · Note Wall"</p>
              <h1>"Note Wall"</h1>
              <p class="lede">"Leave a note. It shows below. Count lives in WASM."</p>
              <input type="text" value={self.draft} on:input={self.set_draft} placeholder="Write something..." />
              <button type="button" on:click={self.post}>"Post"</button>
              <p class="count">{"Posted: "}{self.count}</p>
              <div class="wall">
                  <p class="note">{self.wall}</p>
              </div>
          </div>
      }
  }


   <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Note Wall — Nectar on Taubyte</title>
    <style>
      :root {
        --bg: #0f1a14;
        --fg: #f3efe4;
        --muted: #c8d5cb;
        --accent: #e8a317;
        --panel: rgba(0, 0, 0, 0.28);
      }
      * { box-sizing: border-box; }
      html, body { margin: 0; min-height: 100%; }
      body {
        font-family: "Segoe UI", system-ui, sans-serif;
        color: var(--fg);
        background:
          radial-gradient(ellipse 80% 50% at 15% 0%, #2a5a3c 0%, transparent 50%),
          linear-gradient(165deg, var(--bg), #0a120e);
      }
      #app { min-height: 100vh; display: grid; place-items: center; padding: 2rem; }
      .wrap { max-width: 28rem; width: 100%; text-align: center; }
      .eyebrow {
        margin: 0 0 0.75rem;
        font-size: 0.75rem;
        letter-spacing: 0.08em;
        text-transform: uppercase;
        color: var(--muted);
      }
      h1 { margin: 0 0 0.75rem; font-size: clamp(1.8rem, 5vw, 2.6rem); letter-spacing: -0.03em; }
      .lede { color: var(--muted); line-height: 1.5; margin: 0 0 1.5rem; }
      input[type="text"] {
        width: 100%;
        margin-bottom: 0.75rem;
        padding: 0.75rem 0.9rem;
        border: 1px solid rgba(232, 163, 23, 0.25);
        border-radius: 0.3rem;
        background: var(--panel);
        color: var(--fg);
        font-size: 1rem;
      }
      button {
        padding: 0.7rem 1.2rem;
        border: 0;
        border-radius: 0.25rem;
        background: var(--accent);
        color: #111;
        font-size: 1rem;
        font-weight: 700;
        cursor: pointer;
      }
      .count { margin: 1.25rem 0 0.75rem; font-weight: 700; font-variant-numeric: tabular-nums; }
      .wall {
        margin-top: 0.5rem;
        padding: 1rem;
        background: var(--panel);
        border: 1px solid rgba(232, 163, 23, 0.25);
        border-radius: 0.35rem;
        text-align: left;
        min-height: 3rem;
      }
      .note { margin: 0; color: var(--fg); }
      .boot-error {
        max-width: 36rem;
        padding: 1rem 1.1rem;
        background: #3a1515;
        border: 1px solid #a33;
        border-radius: 0.3rem;
        color: #fcc;
        font-family: ui-monospace, monospace;
        font-size: 0.8rem;
        white-space: pre-wrap;
      }
    </style>
  </head>
  <body>
    <div id="app">Loading Nectar WASM…</div>
    <script type="module">
      async function instantiateWithStubs(wasmUrl, wasmImports, NectarRuntime) {
        const memory = new WebAssembly.Memory({ initial: 256, maximum: 1024 });
        const merged = { env: { memory } };
        for (const [ns, fns] of Object.entries(wasmImports)) {
          merged[ns] = { ...fns };
        }
        const bytes = await fetch(wasmUrl).then((r) => {
          if (!r.ok) throw new Error(`fetch ${wasmUrl} failed: ${r.status}`);
          return r.arrayBuffer();
        });
        const module = await WebAssembly.compile(bytes);
        const stub = () => 0;
        for (const imp of WebAssembly.Module.imports(module)) {
          if (imp.kind === 'memory') {
            if (!merged[imp.module]) merged[imp.module] = {};
            if (!merged[imp.module][imp.name]) merged[imp.module][imp.name] = memory;
            continue;
          }
          if (imp.kind !== 'function') continue;
          if (!merged[imp.module]) merged[imp.module] = {};
          if (typeof merged[imp.module][imp.name] !== 'function') {
            merged[imp.module][imp.name] = stub;
          }
        }
        const instance = await WebAssembly.instantiate(module, merged);
        NectarRuntime.__memory = memory;
        NectarRuntime.__init(instance);
        if (instance.exports.__init_all) instance.exports.__init_all();
        return instance;
      }
      const root = document.getElementById('app');
      try {
        const { wasmImports, NectarRuntime } = await import('./core.js');
        const instance = await instantiateWithStubs('./app.wasm', wasmImports, NectarRuntime);
        root.textContent = '';
        const rootId = NectarRuntime.__registerElement(root);
        if (typeof instance.exports.App_mount !== 'function') {
          throw new Error('app.wasm has no App_mount export — check app.nectar component name');
        }
        instance.exports.App_mount(rootId);
      } catch (err) {
        root.innerHTML = '';
        const pre = document.createElement('pre');
        pre.className = 'boot-error';
        pre.textContent = String(err && err.stack ? err.stack : err);
        root.appendChild(pre);
      }
    </script>
  </body>
  </html>
```
