<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Free Fire Optimizer</title>
  <style>
    body {
      margin: 0;
      background: #0f172a;
      color: white;
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }
    .container {
      width: min(900px, 90%);
      background: #111827;
      border-radius: 18px;
      padding: 24px;
      box-shadow: 0 20px 40px rgba(0,0,0,.35);
    }
    h1 {
      margin-top: 0;
      font-size: 2rem;
    }
    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
    }
    .box {
      background: #1f2937;
      border-radius: 14px;
      padding: 18px;
    }
    label {
      display: block;
      margin-top: 12px;
      color: #cbd5e1;
      font-size: 0.95rem;
    }
    input, select, button {
      width: 100%;
      margin-top: 8px;
      padding: 10px 12px;
      border-radius: 10px;
      border: 1px solid #374151;
      background: #0f172a;
      color: white;
      font-size: 1rem;
    }
    button {
      background: linear-gradient(135deg, #22c55e, #16a34a);
      border: none;
      cursor: pointer;
      font-weight: bold;
      margin-top: 16px;
    }
    .result {
      margin-top: 20px;
      background: #0f172a;
      border-radius: 12px;
      padding: 16px;
      line-height: 1.7;
    }
    @media (max-width: 700px) {
      .grid { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Free Fire Optimizer</h1>

    <div class="grid">
      <div class="box">
        <label>CPU
          <input id="cpu" value="Intel i5 12400F">
        </label>

        <label>GPU
          <input id="gpu" value="RTX 3060">
        </label>

        <label>RAM (GB)
          <input id="ram" type="number" value="16">
        </label>

        <label>Monitor (Hz)
          <select id="monitorHz">
            <option value="60">60Hz</option>
            <option value="90">90Hz</option>
            <option value="120" selected>120Hz</option>
            <option value="144">144Hz</option>
          </select>
        </label>

        <label>Estilo de jogo
          <select id="playstyle">
            <option value="agressivo" selected>Agressivo</option>
            <option value="equilibrado">Equilibrado</option>
            <option value="defensivo">Defensivo</option>
          </select>
        </label>

        <label>Sensibilidade
          <input id="sensibility" type="range" min="0.5" max="2.5" step="0.1" value="1.3">
        </label>
      </div>

      <div class="box">
        <label><input id="colorblind" type="checkbox"> Modo para daltonismo</label>
        <label><input id="highContrast" type="checkbox"> Alto contraste</label>
        <label><input id="subtitles" type="checkbox" checked> Legendas/alertas visuais</label>
        <label><input id="audioCues" type="checkbox" checked> Sinais sonoros</label>

        <button id="generateBtn">Gerar recomendação</button>
      </div>
    </div>

    <div class="result" id="result">
      Configure os dados e gere a recomendação.
    </div>
  </div>

  <script>
    function classifyHardware(gpu, ram) {
      const g = gpu.toLowerCase();
      if (g.includes("gtx 750") || g.includes("intel uhd") || g.includes("rx 550") || ram <= 8) return "baixo";
      if (g.includes("gtx 1050") || g.includes("gtx 1650") || g.includes("rtx 3050") || g.includes("rx 560") || ram <= 16) return "medio";
      return "alto";
    }

    function graphicsByLevel(level) {
      if (level === "baixo") {
        return { resolucao: "1280x720", qualidade: "Baixa", fps: 60, sombras: "Baixas", antiAlias: "Desligado", textura: "Baixa" };
      }
      if (level === "medio") {
        return { resolucao: "1920x1080", qualidade: "Média", fps: 90, sombras: "Médias", antiAlias: "FXAA", textura: "Média" };
      }
      return { resolucao: "1920x1080", qualidade: "Alta", fps: 120, sombras: "Altas", antiAlias: "TAA/FXAA", textura: "Alta" };
    }

    function accessibility(data) {
      let paleta = "Padrão";
      let contraste = "Médio";
      let hud = "HUD padrão";

      if (data.colorblind) {
        paleta = "Azul/ciano + amarelo";
        contraste = "Alto";
        hud = "HUD com contraste reforçado";
      }

      if (data.highContrast) {
        hud = "HUD em alto contraste";
      }

      return {
        paleta,
        contraste,
        hud,
        legendas: data.subtitles ? "Ativadas" : "Desativadas",
        sons: data.audioCues ? "Ativados" : "Desativados"
      };
    }

    function controls(playstyle, sensitivity) {
      if (playstyle === "agressivo") {
        return { sensibilidade: (sensitivity * 1.15).toFixed(2), estilo: "Rápido e agressivo" };
      }
      if (playstyle === "defensivo") {
        return { sensibilidade: (sensitivity * 0.9).toFixed(2), estilo: "Estável e preciso" };
      }
      return { sensibilidade: sensitivity.toFixed(2), estilo: "Equilibrado" };
    }

    document.getElementById("generateBtn").addEventListener("click", () => {
      const data = {
        cpu: document.getElementById("cpu").value,
        gpu: document.getElementById("gpu").value,
        ram: Number(document.getElementById("ram").value || 8),
        monitorHz: Number(document.getElementById("monitorHz").value || 60),
        playstyle: document.getElementById("playstyle").value,
        sensibility: Number(document.getElementById("sensibility").value || 1),
        colorblind: document.getElementById("colorblind").checked,
        highContrast: document.getElementById("highContrast").checked,
        subtitles: document.getElementById("subtitles").checked,
        audioCues: document.getElementById("audioCues").checked
      };

      const level = classifyHardware(data.gpu, data.ram);
      const graphics = graphicsByLevel(level);
      const access = accessibility(data);
      const ctrl = controls(data.playstyle, data.sensibility);

      document.getElementById("result").innerHTML = `
        <strong>Hardware:</strong> ${data.cpu} / ${data.gpu} / ${data.ram} GB / ${data.monitorHz}Hz<br>
        <strong>Estilo:</strong> ${data.playstyle}<br>
        <strong>Gráficos:</strong> ${graphics.qualidade} | Resolução ${graphics.resolucao} | FPS meta ${graphics.fps}<br>
        <strong>Sombras:</strong> ${graphics.sombras} | Anti-aliasing ${graphics.antiAlias} | Textura ${graphics.textura}<br>
        <strong>Acessibilidade:</strong> Paleta ${access.paleta} | Contraste ${access.contraste} | HUD ${access.hud}<br>
        <strong>Legenda:</strong> ${access.legendas} | Alertas sonoros ${access.sons}<br>
        <strong>Controles:</strong> Sensibilidade ${ctrl.sensibilidade} | Estilo ${ctrl.estilo}
      `;
    });
  </script>
</body>
</html>
