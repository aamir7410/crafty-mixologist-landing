# Crafty Mixologist — Landing Page (single-file)

This repository file contains a complete, single-file HTML/CSS/JavaScript landing page you can use to capture email addresses from visitors coming from social media and paid ads.

Instructions for a beginner:

- Save the HTML code below into a file named `index.html` inside your project folder.
- Open `index.html` in a web browser to view and test the page.
- The form stores subscriber emails locally in your browser (Local Storage). You can export subscribers to a CSV using the "Export CSV" button.
- If you want to forward signups to a server or webhook (for Mailchimp, Zapier, etc.), replace the placeholder `WEBHOOK_URL` in the JavaScript with your endpoint.

---

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Crafty Mixologist — Get 10% Off</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--accent:#f59e0b;--muted:#94a3b8;--white:#f8fafc}
    *{box-sizing:border-box}
    body{margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial;background:linear-gradient(180deg,#071028 0%,#081126 100%);color:var(--white);min-height:100vh;display:flex;align-items:center;justify-content:center;padding:24px}
    .card{background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(255,255,255,0.01));backdrop-filter:blur(6px);border:1px solid rgba(255,255,255,0.04);max-width:900px;width:100%;padding:32px;border-radius:12px;display:grid;grid-template-columns:1fr 360px;gap:28px}
    .brand{display:flex;flex-direction:column;gap:14px}
    .logo{font-weight:700;letter-spacing:0.6px;font-size:28px;color:var(--accent)}
    h1{margin:0;font-size:28px}
    p.lead{margin:0;color:var(--muted)}
    .features{display:flex;flex-direction:column;gap:10px;margin-top:14px}
    .feature{display:flex;gap:10px;align-items:flex-start}
    .dot{width:10px;height:10px;border-radius:50%;background:var(--accent);margin-top:6px}
    form{display:flex;flex-direction:column;gap:10px}
    input[type="email"]{padding:12px 14px;border-radius:8px;border:1px solid rgba(255,255,255,0.06);background:transparent;color:var(--white);outline:none}
    button.cta{background:var(--accent);color:#081126;border:none;padding:12px 14px;border-radius:8px;font-weight:700;cursor:pointer}
    .muted{color:var(--muted);font-size:13px}
    .right{background:linear-gradient(180deg,rgba(255,255,255,0.015),transparent);padding:20px;border-radius:10px;border:1px solid rgba(255,255,255,0.03)}
    .thankyou{display:none}
    .links{display:flex;gap:10px;margin-top:12px}
    a.store{color:var(--white);text-decoration:none;border:1px solid rgba(255,255,255,0.04);padding:8px 10px;border-radius:8px;background:transparent}
    footer{margin-top:18px;color:var(--muted);font-size:12px}
    @media (max-width:880px){.card{grid-template-columns:1fr;}
      .right{order:-1}}
  </style>
</head>
<body>
  <main class="card">
    <section class="brand">
      <div>
        <div class="logo">Crafty Mixologist</div>
        <h1>Elevate your cocktails at home</h1>
        <p class="lead">Premium bar tools, mixers, and cocktail kits — curated for craft at-home mixologists.</p>
      </div>

      <div class="features">
        <div class="feature"><span class="dot"></span><div><strong>Free guides</strong><div class="muted">Curated recipes and tips</div></div></div>
        <div class="feature"><span class="dot"></span><div><strong>Quality gear</strong><div class="muted">Tools used by professionals</div></div></div>
        <div class="feature"><span class="dot"></span><div><strong>Shop on Amazon & Etsy</strong><div class="muted">Fast & trusted checkout</div></div></div>
      </div>

      <footer>
        <div class="muted">We respect your privacy — no spam. By subscribing you agree to receive occasional emails.</div>
      </footer>
    </section>

    <aside class="right">
      <div id="formWrap">
        <h2 style="margin:0 0 8px 0">Get 10% off your first order</h2>
        <p class="muted" style="margin:0 0 12px 0">Join the list for exclusive deals and cocktail recipes.</p>

        <form id="signupForm">
          <input id="email" type="email" placeholder="Your email address" required />
          <label style="display:flex;align-items:center;gap:8px;font-size:13px;color:var(--muted)"><input id="consent" type="checkbox" required /> I agree to receive emails</label>
          <button class="cta" type="submit">Get 10% Off</button>
        </form>

        <div class="thankyou" id="thankyou">
          <h3 style="margin:0 0 8px 0">Thanks — you're in!</h3>
          <p class="muted">Check your inbox for the discount code.</p>
        </div>

        <div style="margin-top:12px;display:flex;gap:8px;flex-wrap:wrap">
          <a class="store" href="https://www.amazon.com" target="_blank" rel="noopener">Amazon Store</a>
          <a class="store" href="https://www.etsy.com" target="_blank" rel="noopener">Etsy Shop</a>
        </div>

        <div style="margin-top:12px;display:flex;gap:8px">
          <button id="exportBtn" class="store" style="cursor:pointer">Export CSV</button>
          <button id="clearBtn" class="store" style="cursor:pointer">Clear Local Copies</button>
        </div>
      </div>
    </aside>
  </main>

  <script>
    const form = document.getElementById('signupForm');
    const emailInput = document.getElementById('email');
    const thankyou = document.getElementById('thankyou');
    const exportBtn = document.getElementById('exportBtn');
    const clearBtn = document.getElementById('clearBtn');

    const STORAGE_KEY = 'crafty_mixologist_subscribers_v1';

    function loadSubscribers(){
      try{ return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]') }catch(e){return []}
    }

    function saveSubscriber(obj){
      const list = loadSubscribers();
      list.push(obj);
      localStorage.setItem(STORAGE_KEY, JSON.stringify(list));
    }

    function validateEmail(email){
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }

    form.addEventListener('submit', (e)=>{
      e.preventDefault();
      const email = emailInput.value.trim();
      const consent = document.getElementById('consent').checked;
      if(!validateEmail(email)){
        alert('Please enter a valid email address.');
        return;
      }
      if(!consent){
        alert('Please agree to receive emails.');
        return;
      }

      const subscriber = {email, date: new Date().toISOString()};
      saveSubscriber(subscriber);

      // If you have a webhook (Zapier, Make, server), set the URL below and uncomment the fetch.
      const WEBHOOK_URL = '';
      if(WEBHOOK_URL){
        fetch(WEBHOOK_URL, {method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(subscriber)}).catch(()=>{/* ignore errors for now */});
      }

      form.style.display = 'none';
      thankyou.style.display = 'block';
    });

    exportBtn.addEventListener('click', ()=>{
      const rows = loadSubscribers();
      if(rows.length===0){ alert('No subscribers saved yet.'); return; }
      const csv = ['email,date'].concat(rows.map(r=>`${r.email},${r.date}`)).join('\n');
      const blob = new Blob([csv],{type:'text/csv'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = 'subscribers.csv'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
    });

    clearBtn.addEventListener('click', ()=>{
      if(confirm('Clear all locally saved subscribers?')){ localStorage.removeItem(STORAGE_KEY); alert('Cleared.'); }
    });
  </script>
</body>
</html>
```

---

Notes for next steps (suggested):

- Replace `WEBHOOK_URL` with your Zapier / Make / server webhook to forward signups.
- Use an email marketing service (Mailchimp, Klaviyo) to manage campaigns and deliver the discount code.
- If you want, I can produce a ready-to-deploy `index.html` file directly in the repo and wire a webhook example.
# Crafty Mixologist — Landing Page (single-file)

This repository file contains a complete, single-file HTML/CSS/JavaScript landing page you can use to capture email addresses from visitors coming from social media and paid ads.

Instructions for a beginner:

- Save the HTML code below into a file named `index.html` inside your project folder.
- Open `index.html` in a web browser to view and test the page.
- The form stores subscriber emails locally in your browser (Local Storage). You can export subscribers to a CSV using the "Export CSV" button.
- If you want to forward signups to a server or webhook (for Mailchimp, Zapier, etc.), replace the placeholder `WEBHOOK_URL` in the JavaScript with your endpoint.

---

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Crafty Mixologist — Get 10% Off</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--accent:#f59e0b;--muted:#94a3b8;--white:#f8fafc}
    *{box-sizing:border-box}
    body{margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial;background:linear-gradient(180deg,#071028 0%,#081126 100%);color:var(--white);min-height:100vh;display:flex;align-items:center;justify-content:center;padding:24px}
    .card{background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(255,255,255,0.01));backdrop-filter:blur(6px);border:1px solid rgba(255,255,255,0.04);max-width:900px;width:100%;padding:32px;border-radius:12px;display:grid;grid-template-columns:1fr 360px;gap:28px}
    .brand{display:flex;flex-direction:column;gap:14px}
    .logo{font-weight:700;letter-spacing:0.6px;font-size:28px;color:var(--accent)}
    h1{margin:0;font-size:28px}
    p.lead{margin:0;color:var(--muted)}
    .features{display:flex;flex-direction:column;gap:10px;margin-top:14px}
    .feature{display:flex;gap:10px;align-items:flex-start}
    .dot{width:10px;height:10px;border-radius:50%;background:var(--accent);margin-top:6px}
    form{display:flex;flex-direction:column;gap:10px}
    input[type="email"]{padding:12px 14px;border-radius:8px;border:1px solid rgba(255,255,255,0.06);background:transparent;color:var(--white);outline:none}
    button.cta{background:var(--accent);color:#081126;border:none;padding:12px 14px;border-radius:8px;font-weight:700;cursor:pointer}
    .muted{color:var(--muted);font-size:13px}
    .right{background:linear-gradient(180deg,rgba(255,255,255,0.015),transparent);padding:20px;border-radius:10px;border:1px solid rgba(255,255,255,0.03)}
    .thankyou{display:none}
    .links{display:flex;gap:10px;margin-top:12px}
    a.store{color:var(--white);text-decoration:none;border:1px solid rgba(255,255,255,0.04);padding:8px 10px;border-radius:8px;background:transparent}
    footer{margin-top:18px;color:var(--muted);font-size:12px}
    @media (max-width:880px){.card{grid-template-columns:1fr;}
      .right{order:-1}}
  </style>
</head>
<body>
  <main class="card">
    <section class="brand">
      <div>
        <div class="logo">Crafty Mixologist</div>
        <h1>Elevate your cocktails at home</h1>
        <p class="lead">Premium bar tools, mixers, and cocktail kits — curated for craft at-home mixologists.</p>
      </div>

      <div class="features">
        <div class="feature"><span class="dot"></span><div><strong>Free guides</strong><div class="muted">Curated recipes and tips</div></div></div>
        <div class="feature"><span class="dot"></span><div><strong>Quality gear</strong><div class="muted">Tools used by professionals</div></div></div>
        <div class="feature"><span class="dot"></span><div><strong>Shop on Amazon & Etsy</strong><div class="muted">Fast & trusted checkout</div></div></div>
      </div>

      <footer>
        <div class="muted">We respect your privacy — no spam. By subscribing you agree to receive occasional emails.</div>
      </footer>
    </section>

    <aside class="right">
      <div id="formWrap">
        <h2 style="margin:0 0 8px 0">Get 10% off your first order</h2>
        <p class="muted" style="margin:0 0 12px 0">Join the list for exclusive deals and cocktail recipes.</p>

        <form id="signupForm">
          <input id="email" type="email" placeholder="Your email address" required />
          <label style="display:flex;align-items:center;gap:8px;font-size:13px;color:var(--muted)"><input id="consent" type="checkbox" required /> I agree to receive emails</label>
          <button class="cta" type="submit">Get 10% Off</button>
        </form>

        <div class="thankyou" id="thankyou">
          <h3 style="margin:0 0 8px 0">Thanks — you're in!</h3>
          <p class="muted">Check your inbox for the discount code.</p>
        </div>

        <div style="margin-top:12px;display:flex;gap:8px;flex-wrap:wrap">
          <a class="store" href="https://www.amazon.com" target="_blank" rel="noopener">Amazon Store</a>
          <a class="store" href="https://www.etsy.com" target="_blank" rel="noopener">Etsy Shop</a>
        </div>

        <div style="margin-top:12px;display:flex;gap:8px">
          <button id="exportBtn" class="store" style="cursor:pointer">Export CSV</button>
          <button id="clearBtn" class="store" style="cursor:pointer">Clear Local Copies</button>
        </div>
      </div>
    </aside>
  </main>

  <script>
    const form = document.getElementById('signupForm');
    const emailInput = document.getElementById('email');
    const thankyou = document.getElementById('thankyou');
    const exportBtn = document.getElementById('exportBtn');
    const clearBtn = document.getElementById('clearBtn');

    const STORAGE_KEY = 'crafty_mixologist_subscribers_v1';

    function loadSubscribers(){
      try{ return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]') }catch(e){return []}
    }

    function saveSubscriber(obj){
      const list = loadSubscribers();
      list.push(obj);
      localStorage.setItem(STORAGE_KEY, JSON.stringify(list));
    }

    function validateEmail(email){
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }

    form.addEventListener('submit', (e)=>{
      e.preventDefault();
      const email = emailInput.value.trim();
      const consent = document.getElementById('consent').checked;
      if(!validateEmail(email)){
        alert('Please enter a valid email address.');
        return;
      }
      if(!consent){
        alert('Please agree to receive emails.');
        return;
      }

      const subscriber = {email, date: new Date().toISOString()};
      saveSubscriber(subscriber);

      // If you have a webhook (Zapier, Make, server), set the URL below and uncomment the fetch.
      const WEBHOOK_URL = '';
      if(WEBHOOK_URL){
        fetch(WEBHOOK_URL, {method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(subscriber)}).catch(()=>{/* ignore errors for now */});
      }

      form.style.display = 'none';
      thankyou.style.display = 'block';
    });

    exportBtn.addEventListener('click', ()=>{
      const rows = loadSubscribers();
      if(rows.length===0){ alert('No subscribers saved yet.'); return; }
      const csv = ['email,date'].concat(rows.map(r=>`${r.email},${r.date}`)).join('\n');
      const blob = new Blob([csv],{type:'text/csv'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = 'subscribers.csv'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
    });

    clearBtn.addEventListener('click', ()=>{
      if(confirm('Clear all locally saved subscribers?')){ localStorage.removeItem(STORAGE_KEY); alert('Cleared.'); }
    });
  </script>
</body>
</html>
```

---

Notes for next steps (suggested):

- Replace `WEBHOOK_URL` with your Zapier / Make / server webhook to forward signups.
- Use an email marketing service (Mailchimp, Klaviyo) to manage campaigns and deliver the discount code.
- If you want, I can produce a ready-to-deploy `index.html` file directly in the repo and wire a webhook example.
