
<html lang="de">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Valentine</title>
  <style>
    :root { --pad: 24px; }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      background: radial-gradient(circle at top, #ffe6ef, #fff);
      color: #1b1b1b;
      overflow: hidden;
    }

    .card {
      width: min(720px, calc(100vw - 2*var(--pad)));
      background: rgba(255,255,255,0.85);
      backdrop-filter: blur(6px);
      border: 1px solid rgba(0,0,0,0.08);
      border-radius: 18px;
      padding: clamp(20px, 4vw, 36px);
      box-shadow: 0 20px 60px rgba(0,0,0,0.08);
      text-align: center;
      position: relative;
    }

    h1 {
      font-size: clamp(26px, 4vw, 42px);
      margin: 0 0 18px 0;
      letter-spacing: 0.2px;
    }

    .subtitle {
      margin: 0 0 24px 0;
      opacity: 0.75;
      font-size: 14px;
    }

    .btnrow {
      display: inline-flex;
      gap: 14px;
      align-items: center;
      justify-content: center;
      margin-top: 8px;
    }

    button {
      appearance: none;
      border: 0;
      border-radius: 999px;
      padding: 14px 26px;
      font-size: 16px;
      font-weight: 650;
      cursor: pointer;
      transition: transform 120ms ease, box-shadow 120ms ease;
      user-select: none;
    }

    button:active { transform: scale(0.98); }

    #yesBtn {
      background: #ff2d72;
      color: white;
      box-shadow: 0 10px 22px rgba(255,45,114,0.25);
    }
    #yesBtn:hover { box-shadow: 0 14px 30px rgba(255,45,114,0.30); }

    #noBtn {
      background: #f1f3f5;
      color: #1b1b1b;
      box-shadow: 0 10px 22px rgba(0,0,0,0.08);
      position: absolute; /* wird per JS bewegt */
      left: 50%;
      top: 62%;
      transform: translate(-50%, -50%);
    }

    .screen {
      width: 100%;
    }

    .hidden { display: none; }

    .gifwrap {
      margin-top: 18px;
      display: grid;
      place-items: center;
    }

    img {
      max-width: min(520px, 100%);
      border-radius: 16px;
      border: 1px solid rgba(0,0,0,0.08);
      box-shadow: 0 18px 40px rgba(0,0,0,0.10);
    }

    .hearts {
      position: fixed;
      inset: 0;
      pointer-events: none;
      opacity: 0.25;
      background-image:
        radial-gradient(circle at 12% 20%, #ff2d72 0 2px, transparent 3px),
        radial-gradient(circle at 80% 25%, #ff2d72 0 2px, transparent 3px),
        radial-gradient(circle at 30% 78%, #ff2d72 0 2px, transparent 3px),
        radial-gradient(circle at 70% 82%, #ff2d72 0 2px, transparent 3px);
      background-size: 260px 260px;
      animation: float 12s linear infinite;
    }

    @keyframes float {
      from { transform: translateY(0); }
      to   { transform: translateY(-120px); }
    }
  </style>
</head>

<body>
  <div class="hearts" aria-hidden="true"></div>

  <div class="card" id="stage">
    <!-- Screen 1 -->
    <section class="screen" id="screen1">
      <h1>Cherie will you be my valentine?</h1>
      <p class="subtitle">Bitte eine Option wählen 🙂</p>

      <div class="btnrow" aria-label="Antwort Buttons">
        <button id="yesBtn" type="button">Yes</button>
        <!-- No-Button wird absolut positioniert und bewegt -->
        <button id="noBtn" type="button" tabindex="-1" aria-label="No">No</button>
      </div>
    </section>

    <!-- Screen 2 -->
    <section class="screen hidden" id="screen2">
      <h1>Cherie will you be my valentine?</h1>
      <div class="gifwrap">
        <img
          id="valentineGif"
          src="https://i.pinimg.com/originals/4c/ea/03/4cea031fa751e7658f5e0355def16f29.gif"
          alt="Valentine GIF"
          loading="lazy"
        />
      </div>
    </section>
  </div>

  <script>
    (function () {
      const stage = document.getElementById("stage");
      const screen1 = document.getElementById("screen1");
      const screen2 = document.getElementById("screen2");
      const yesBtn = document.getElementById("yesBtn");
      const noBtn = document.getElementById("noBtn");

      // Mindestabstand Maus <-> No-Button (Pixel)
      const MIN_DIST = 140;

      // Hilfsfunktionen
      function clamp(v, min, max) { return Math.max(min, Math.min(max, v)); }

      function stageRect() { return stage.getBoundingClientRect(); }

      function getBtnSize() {
        return { w: noBtn.offsetWidth || 100, h: noBtn.offsetHeight || 44 };
      }

      // Setzt absolute Position innerhalb der Card (stage)
      function setNoPos(x, y) {
        const r = stageRect();
        const { w, h } = getBtnSize();

        // innerhalb der Card bleiben
        const maxX = r.width - w;
        const maxY = r.height - h;

        const nx = clamp(x, 0, maxX);
        const ny = clamp(y, 0, maxY);

        noBtn.style.transform = "translate(0, 0)";
        noBtn.style.left = nx + "px";
        noBtn.style.top = ny + "px";
      }

      function currentNoCenter() {
        const b = noBtn.getBoundingClientRect();
        return { cx: b.left + b.width / 2, cy: b.top + b.height / 2 };
      }

      function randomPosAwayFrom(mouseX, mouseY) {
        const r = stageRect();
        const { w, h } = getBtnSize();

        // Mauskoordinaten in Stage-Koordinaten umrechnen
        const mx = mouseX - r.left;
        const my = mouseY - r.top;

        const tries = 80;
        for (let i = 0; i < tries; i++) {
          const x = Math.random() * (r.width - w);
          const y = Math.random() * (r.height - h);
          const cx = x + w / 2;
          const cy = y + h / 2;
          const d = Math.hypot(cx - mx, cy - my);
          if (d >= MIN_DIST) return { x, y };
        }

        // Fallback: entfernteste Ecke
        const corners = [
          { x: 0, y: 0 },
          { x: r.width - w, y: 0 },
          { x: 0, y: r.height - h },
          { x: r.width - w, y: r.height - h }
        ];
        corners.sort((a, b) => {
          const da = Math.hypot((a.x + w/2) - mx, (a.y + h/2) - my);
          const db = Math.hypot((b.x + w/2) - mx, (b.y + h/2) - my);
          return db - da;
        });
        return corners[0];
      }

      function evadeIfTooClose(e) {
        // Nur solange Screen 1 sichtbar ist
        if (screen1.classList.contains("hidden")) return;

        const { cx, cy } = currentNoCenter();
        const d = Math.hypot(cx - e.clientX, cy - e.clientY);

        if (d < MIN_DIST) {
          const p = randomPosAwayFrom(e.clientX, e.clientY);
          setNoPos(p.x, p.y);
        }
      }

      // Initiale Position etwas rechts unten setzen (innerhalb der Card)
      function initialPlace() {
        const r = stageRect();
        const { w, h } = getBtnSize();
        setNoPos(r.width / 2 + 70, r.height * 0.62 - h / 2);
      }

      // Events
      document.addEventListener("mousemove", evadeIfTooClose, { passive: true });
      document.addEventListener("pointermove", evadeIfTooClose, { passive: true });

      // No darf nicht per Tastatur "gewonnen" werden
      noBtn.addEventListener("focus", (e) => {
        noBtn.blur();
        const p = randomPosAwayFrom(window.innerWidth / 2, window.innerHeight / 2);
        setNoPos(p.x, p.y);
      });

      // Auch bei direkten Interaktionen weg
      noBtn.addEventListener("mouseenter", (e) => evadeIfTooClose(e));
      noBtn.addEventListener("pointerdown", (e) => {
        e.preventDefault();
        const p = randomPosAwayFrom(e.clientX, e.clientY);
        setNoPos(p.x, p.y);
      });

      yesBtn.addEventListener("click", () => {
        screen1.classList.add("hidden");
        screen2.classList.remove("hidden");
      });

      window.addEventListener("resize", initialPlace);

      // Start
      // (kleiner Timeout, damit Layout/Fonts sicher berechnet sind)
      setTimeout(initialPlace, 0);
    })();
  </script>
</body>
</html>
