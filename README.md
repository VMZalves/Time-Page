[index.html](https://github.com/user-attachments/files/30429380/index.html)
<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Hora de Brasília + QR PIX</title>
  <style>
    :root{
      --bg:#0f1724;
      --card:#0b1220;
      --accent:#06b6d4;
      --text:#e6eef6;
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      min-height:100vh;
      display:flex;
      align-items:center;
      justify-content:center;
      background:linear-gradient(180deg,#061021 0%, #071224 100%);
      font-family:system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      color:var(--text);
      padding:24px;
    }
    .card{
      background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border:1px solid rgba(255,255,255,0.04);
      padding:28px;
      width:100%;
      max-width:560px;
      border-radius:12px;
      box-shadow: 0 6px 30px rgba(2,6,23,0.6);
      text-align:center;
    }
    #datetime{margin-bottom:20px}
    #date{font-size:1.05rem; opacity:0.9; margin-bottom:6px; text-transform:capitalize}
    #time{font-size:2.25rem; font-weight:700; color:var(--accent); letter-spacing:1px}
    .pix-area{
      margin-top:18px;
      display:flex;
      flex-direction:column;
      gap:12px;
      align-items:center;
    }
    .controls{
      display:flex;
      gap:8px;
      width:100%;
      max-width:480px;
    }
    input[type="text"]{
      flex:1;
      padding:10px 12px;
      border-radius:8px;
      border:1px solid rgba(255,255,255,0.06);
      background:rgba(255,255,255,0.02);
      color:var(--text);
      font-size:1rem;
    }
    button{
      padding:10px 14px;
      border-radius:8px;
      border: none;
      background:var(--accent);
      color:#022;
      font-weight:700;
      cursor:pointer;
    }
    #qrcode{padding:14px; border-radius:8px; background:rgba(255,255,255,0.01); display:flex; align-items:center; justify-content:center; min-height:170px; min-width:170px}
    #qrcode img{max-width:100%; height:auto; display:block}
    #pixText{font-size:0.95rem; opacity:0.9; word-break:break-all}
    #download{color:var(--text); text-decoration:none; background:transparent; border:1px solid rgba(255,255,255,0.06); padding:8px 10px; border-radius:8px}
    .hint{font-size:0.85rem; opacity:0.8}
    @media (max-width:520px){
      #time{font-size:1.6rem}
      .controls{flex-direction:column}
      button{width:100%}
    }
  </style>
</head>
<body>
  <main class="card" role="main" aria-labelledby="title">
    <h1 id="title" style="margin:0 0 12px 0; font-size:1.1rem;">Relógio (Horário de Brasília) + QR PIX</h1>

    <div id="datetime" aria-live="polite">
      <div id="date">—</div>
      <div id="time">—</div>
    </div>

    <section class="pix-area" aria-label="PIX QR code">
      <div class="controls" role="form" aria-label="Gerador de QR PIX">
        <input id="pixKey" type="text" inputmode="text" placeholder="Cole a chave PIX aqui (CPF, e-mail, telefone ou chave aleatória)" aria-label="Chave PIX"/>
        <button id="generate">Gerar QR</button>
      </div>

      <div id="qrcode" aria-hidden="false" title="QR code da chave PIX">
        <!-- QR aparecerá aqui -->
        <span class="hint">QR aparecerá aqui após gerar</span>
      </div>

      <div style="display:flex;gap:8px;align-items:center;flex-wrap:wrap;justify-content:center">
        <a id="download" href="#" download="pix-qrcode.png" style="display:none">Baixar QR</a>
        <button id="copyKey" style="background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--text);padding:8px;border-radius:8px">Copiar chave</button>
      </div>

      <div id="pixText" aria-live="polite"></div>
      <div class="hint" style="margin-top:6px">Observação: este QR gera a imagem com o texto da chave PIX. Se quiser um QR no padrão EMV (QR estático do PIX com payload completo), posso implementar — envie os dados que deseja embutir (nome, cidade, valor, tipo de chave).</div>
    </section>
  </main>

  <!-- Biblioteca para gerar QR (qrcode.js via jsDelivr) -->
  <script src="https://cdn.jsdelivr.net/npm/qrcode@1.5.1/build/qrcode.min.js"></script>
  <script>
    // Atualiza data/hora no fuso de Brasília (America/Sao_Paulo)
    function updateDateTime(){
      const now = new Date();
      const optionsDate = { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric', timeZone: 'America/Sao_Paulo' };
      const optionsTime = { hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false, timeZone: 'America/Sao_Paulo' };
      document.getElementById('date').textContent = now.toLocaleDateString('pt-BR', optionsDate);
      document.getElementById('time').textContent = now.toLocaleTimeString('pt-BR', optionsTime);
    }
    updateDateTime();
    setInterval(updateDateTime, 1000);

    // Gerar QR a partir do texto (chave PIX)
    const generateBtn = document.getElementById('generate');
    const qrcodeDiv = document.getElementById('qrcode');
    const pixText = document.getElementById('pixText');
    const downloadLink = document.getElementById('download');
    const copyBtn = document.getElementById('copyKey');

    async function generateQR(){
      const key = document.getElementById('pixKey').value.trim();
      if(!key){
        alert('Digite ou cole a chave PIX antes de gerar o QR.');
        return;
      }
      pixText.textContent = 'Chave PIX: ' + key;
      qrcodeDiv.innerHTML = ''; // limpa

      try{
        // Gera DataURL do QR (usa a biblioteca QRCode)
        const dataUrl = await QRCode.toDataURL(key, { width: 300, margin: 1 });
        const img = document.createElement('img');
        img.alt = 'QR code da chave PIX';
        img.src = dataUrl;
        qrcodeDiv.appendChild(img);

        // Habilita download
        downloadLink.href = dataUrl;
        downloadLink.style.display = 'inline-block';
      }catch(err){
        console.error(err);
        alert('Erro ao gerar QR: ' + (err && err.message ? err.message : err));
      }
    }

    generateBtn.addEventListener('click', generateQR);
    document.getElementById('pixKey').addEventListener('keyup', function(e){
      if(e.key === 'Enter') generateQR();
    });

    copyBtn.addEventListener('click', async () => {
      const key = document.getElementById('pixKey').value.trim();
      if(!key){
        alert('Não há chave para copiar.');
        return;
      }
      try{
        await navigator.clipboard.writeText(key);
        copyBtn.textContent = 'Copiado ✓';
        setTimeout(()=> copyBtn.textContent = 'Copiar chave', 1500);
      }catch(e){
        alert('Não foi possível copiar: ' + e);
      }
    });
  </script>
</body>
</html>
