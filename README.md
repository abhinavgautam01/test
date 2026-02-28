
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AI Security Auditor</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; }

    body {
      margin: 0;
      font-family: 'Courier New', Courier, monospace;
      background: linear-gradient(135deg, #0d0d1a 0%, #0a1628 100%);
      color: #c8ffcc;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 40px 16px;
    }

    .card {
      background: rgba(0, 255, 80, 0.04);
      border: 1px solid #00ff5540;
      border-radius: 12px;
      padding: 36px 40px;
      max-width: 720px;
      width: 100%;
      box-shadow: 0 0 30px #00ff2210;
    }

    h1 {
      margin: 0 0 6px;
      font-size: 1.9rem;
      color: #00ff88;
      text-shadow: 0 0 12px #00ff88aa;
    }

    .subtitle {
      color: #7aff9a88;
      font-size: 0.88rem;
      margin-bottom: 28px;
    }

    label {
      display: block;
      font-size: 0.85rem;
      color: #7aff9a;
      margin-bottom: 6px;
    }

    .input-row {
      display: flex;
      gap: 10px;
    }

    input[type="text"] {
      flex: 1;
      background: #050d15;
      border: 1px solid #00ff5550;
      color: #c8ffcc;
      font-family: inherit;
      font-size: 0.92rem;
      padding: 10px 14px;
      border-radius: 6px;
      outline: none;
      transition: border-color 0.2s;
    }

    input[type="text"]:focus {
      border-color: #00ff88;
    }

    button {
      background: #00ff55;
      color: #050d15;
      font-family: inherit;
      font-weight: 700;
      font-size: 0.92rem;
      border: none;
      border-radius: 6px;
      padding: 10px 22px;
      cursor: pointer;
      transition: background 0.2s, box-shadow 0.2s;
      white-space: nowrap;
    }

    button:hover:not(:disabled) {
      background: #00ffaa;
      box-shadow: 0 0 12px #00ff8880;
    }

    button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }

    #status {
      margin-top: 16px;
      font-size: 0.82rem;
      color: #7aff9a88;
      min-height: 18px;
    }

    #output {
      margin-top: 24px;
      display: none;
    }

    #output h2 {
      font-size: 1rem;
      color: #00ff88;
      margin: 0 0 8px;
    }

    pre {
      background: #050d15;
      border: 1px solid #00ff5530;
      border-radius: 8px;
      padding: 18px;
      white-space: pre-wrap;
      word-break: break-word;
      font-size: 0.82rem;
      color: #c8ffcc;
      max-height: 520px;
      overflow-y: auto;
      line-height: 1.55;
    }

    .error { color: #ff5555; }

    .info-box {
      margin-top: 28px;
      background: rgba(0,180,255,0.05);
      border: 1px solid #00aaff30;
      border-radius: 8px;
      padding: 16px 20px;
      font-size: 0.8rem;
      color: #88ccff;
      line-height: 1.6;
    }

    .info-box strong { color: #aaddff; }
  </style>
</head>
<body>
  <div class="card">
    <h1>🔍 AI Security Auditor</h1>
    <p class="subtitle">Automated deep-scan powered by an LLM agent + Ollama neural weights</p>

    <form id="auditForm" autocomplete="off">
      <label for="githubUrl">GitHub Repository URL</label>
      <div class="input-row">
        <input
          type="text"
          id="githubUrl"
          placeholder="https://github.com/username/repo"
          required
        />
        <button type="submit" id="submitBtn">Audit →</button>
      </div>
    </form>

    <div id="status"></div>

    <div id="output">
      <h2>📋 Audit Report</h2>
      <pre id="reportPre"></pre>
    </div>

    <div class="info-box">
      <strong>How it works:</strong><br />
      The agent clones the repository, reads the README, and installs any required
      neural-weight dependencies before performing the security scan.<br /><br />
      <strong>Internal stack:</strong> Flask agent → <code>Ollama v0.1.33</code>
      (internal network only, not directly reachable by users)
    </div>
  </div>

  <script>
    const form    = document.getElementById('auditForm');
    const urlIn   = document.getElementById('githubUrl');
    const btn     = document.getElementById('submitBtn');
    const status  = document.getElementById('status');
    const output  = document.getElementById('output');
    const pre     = document.getElementById('reportPre');

    function escape(s) {
      return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
    }

    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      const url = urlIn.value.trim();
      if (!url) return;

      btn.disabled = true;
      status.textContent = '⏳ Fetching repository and running audit …';
      output.style.display = 'none';
      pre.innerHTML = '';

      try {
        const resp = await fetch('/analyze', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ url })
        });
        const data = await resp.json();

        if (data.error) {
          status.innerHTML = `<span class="error">❌ ${escape(data.error)}</span>`;
        } else {
          status.textContent = '✅ Audit complete.';
          pre.innerHTML = escape(data.report || '');
          output.style.display = 'block';
        }
      } catch (err) {
        status.innerHTML = `<span class="error">❌ Network error: ${escape(String(err))}</span>`;
      } finally {
        btn.disabled = false;
      }
    });
  </script>
</body>
</html>
