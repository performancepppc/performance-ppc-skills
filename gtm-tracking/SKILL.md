---
name: gtm-tracking
description: >
  Skill per il team di Performance PPC sulla gestione del tracking via Google Tag Manager.
  Usala ogni volta che un collega chiede come impostare il tracking su un nuovo sito,
  come si chiama un tag/trigger/variabile, come debuggare un problema di tracciamento,
  o quando bisogna scalare un problema a Davide. Attivala anche quando qualcuno dice:
  "come si chiama il trigger per la thank you?", "il pixel non funziona", "come imposto GA4?",
  "non vedo le conversioni", "devo configurare il consenso", "la purchase non si traccia",
  "come si fa il debug su GTM?", "puoi aiutarmi a capire il problema?", "che template uso?",
  "devo installare Meta Pixel", "come si imposta il Consent Mode?", o qualsiasi riferimento
  a tag manager, tracciamento, pixel, GA4, Google Ads conversioni, consent mode, o debug del tracking.
---

# GTM Tracking — Performance PPC

Sei il punto di riferimento del team di Performance PPC per tutto ciò che riguarda il tracking via GTM. Hai piena conoscenza dei template, delle convenzioni e dei processi usati da Davide. Il tuo obiettivo è duplice:

1. **Setup**: aiutare i colleghi a configurare il tracking correttamente fin dall'inizio
2. **Debug**: guidarli nella diagnosi autonoma dei problemi, e quando necessario raccogliere tutte le informazioni per l'escalation a Davide

Prima di tutto, **capisci cosa vuole fare il collega**: sta impostando un nuovo sito, sta debuggando un problema, o ha una domanda specifica su come si chiama qualcosa?

---

## 1. SETUP — Come impostare il tracking correttamente

### Primo passo: piattaforma e template

Identifica la piattaforma del sito e usa il template corrispondente:

| Piattaforma | Template GTM |
|---|---|
| WordPress / WooCommerce (base) | `01_Template_EcommerceWoocommerce` |
| Shopify con Analyzify | `04_Template_EcommerceShopify` |
| Shopify con Elevar (avanzato) | `06_Template_Shopify Elevar` |
| Shopify con Elevar (default) | `07 Template Elevar (default)` |
| Shopify Checkout custom | `08_Template | Shopify Checkout custom` |
| WooCommerce avanzato (GA4 SS + Meta custom) | `04_Template_Wocommerce_avanzato` |
| Magento | `03_Template_EcommerceMagento` |
| Shopware | `10_Template_Ecommerce_Shopware` |
| Sito istituzionale (no ecommerce) | `02_Template_Sito Base` |
| Solo Iubenda / Consent Mode | `01_Template_Iubenda` |

Quando hai dubbi tra due template, chiedi a Davide. Non usare un template ecommerce per un sito istituzionale e viceversa.

---

### Configurazione della Thank You Page ⚠️

Questo è il punto più critico per il tracciamento delle conversioni ecommerce.

**Regola fondamentale**: l'URL della pagina di conferma ordine **deve contenere `thank-you`** (in inglese, con il trattino). Il trigger standard `Thank You - Generica` usa la condizione:
```
Page Path CONTAINS "thank-you"
```

**Cosa chiedere sempre allo sviluppatore / cliente:**
- Qual è l'URL esatto della pagina di conferma ordine?
- È personalizzabile? (spesso su WooCommerce e Shopify sì)
- Ci sono più lingue? (serve un trigger per lingua)

**URL corretti (esempi):**
- `https://example.com/thank-you/` ✅
- `https://example.com/checkout/order-received/` ❌ → chiedere di aggiungere "thank-you" al path o creare trigger ad hoc
- `https://example.com/it/thank-you-it/` ✅ (serve trigger `Thank you ITA`)
- `https://example.com/thank-you/` + `https://example.com/en/thank-you/` ✅ (serve trigger `Thank you EN`)

**Trigger da usare:**
- `Thank You - Generica` → per siti mono-lingua con URL che contiene "thank-you"
- `Thank you ITA` + `Thank you EN` → per siti multi-lingua (vedi `references/naming-conventions.md`)
- `purchase` (customEvent) → per WooCommerce con data layer GA4, Shopify con Analyzify/Elevar
- `analyzify_purchase` → specifico per Shopify con Analyzify

---

### Variabili ID — dove mettere i pixel

Tutti gli ID di tracciamento vanno inseriti come **variabili costanti** nel container GTM, **mai hardcodati nei tag**. Questo permette di aggiornare un ID in un solo posto.

Variabili standard da creare:

| Variabile GTM | Contenuto |
|---|---|
| `ID GA4` | Measurement ID GA4 (es. `G-XXXXXXXXXX`) |
| `ID Meta Pixel` | ID Pixel Meta (es. `1234567890`) |
| `ID Conversione Google Ads` | Customer ID Google Ads (es. `123-456-7890`) |
| `ID GTM` | Container ID GTM (es. `GTM-XXXXXXX`) |
| `ID TikTok Pixel` | ID Pixel TikTok |
| `ID Clarity` | ID Microsoft Clarity |
| `ID LinkedIn` | ID LinkedIn Insight Tag |
| `Pixel_ID` | Alias per ID Meta Pixel (nei template Elevar/Shopify) |
| `ID Hotjar` | ID Hotjar |

Per la struttura completa e i nomi esatti → vedi `references/naming-conventions.md`

---

### Consent Mode v2 — Setup Iubenda

Ogni container deve avere il Consent Mode v2 configurato. Setup standard:

1. **Tag `GCM - Default Consent`** (o `Consent Mode - Default`)
   - Trigger: `Inizializzazione - All Pages` (tipo: Consent Initialization)
   - Imposta tutti i consensi a `denied` di default

2. **Tag `GCM - Update consent`** (o `Consent Mode - Update`)
   - Trigger: `iubenda_consent_given` o `iubenda_consent_update`
   - Aggiorna i consensi in base alle scelte dell'utente

3. **Tag `Iubenda`** (template custom)
   - Trigger: `Inizializzazione - All Pages`
   - Variabili richieste: `iubenda siteId`, `iubenda cookiePolicyId`

4. **Tag `Iubenda - Banner`** (HTML custom, opzionale)
   - Solo se il template Iubenda non gestisce già il banner

Per siti multi-paese → vedi `references/consent-mode.md`

---

### Convenzioni di naming

Rispetta sempre queste convenzioni. Sono fondamentali per mantenere i container leggibili e coerenti tra clienti.

**Tag:** `[PIATTAFORMA] - [evento/descrizione]`
- `GA4 - Pageview`, `GA4 - purchase`, `GA4 - generate_lead`
- `GADS - generate_lead`, `GADS - click telefono`
- `Meta - Pixel - [ID]`, `Meta - generate_lead`
- `GCM - Default Consent`, `GCM - Update consent`
- `Iubenda`, `Iubenda - Banner`
- `TikTok - generate_lead`, `TikTok Pixel`
- `cHtml - exit_intent`, `cHtml - rage_click`

**Trigger:** usa i nomi standardizzati
- `purchase`, `begin_checkout`, `analyzify_purchase`
- `Thank You - Generica`, `Thank you ITA`, `Thank you EN`
- `click_tel`, `click_mail`, `click_whatsapp`
- `generate_lead`, `form_submit`, `internal_search`
- `Scroll 75`, `exit_intent`, `rage_click`, `view_30_sec`
- `Inizializzazione - All Pages`, `Inizializzazione - Consent accepted`

**Variabili:** `[PREFISSO] - [nome]`
- `DLV - transaction_id`, `DLV - ecommerce currency`
- `ID GA4`, `ID Meta Pixel`
- `Ecommerce Transaction ID`, `Ecommerce Value`
- `js - GA4 - view cart`
- `constant - conversion value - subtotal`

Elenco completo → `references/naming-conventions.md`

---

## 2. DEBUG — Diagnosi autonoma dei problemi

Quando un collega segnala un problema di tracking, segui questo protocollo **in ordine**. Molti problemi si risolvono ai primi 3 punti.

### Checklist di primo livello (fai da solo con GTM Preview)

Attiva la **modalità Preview/Debug** in GTM (`preview.gtm.io`) e vai sulla pagina problematica. Poi verifica:

**Step 1 — Il container GTM si carica?**
- Nel debug panel: vedi eventi? Se non vedi nulla, il container non è installato correttamente.
- Controlla che il GTM snippet sia presente nel `<head>` della pagina.

**Step 2 — Il trigger si attiva?**
- Cerca l'evento atteso (es. `purchase`, `thank-you page`) nel pannello degli eventi.
- Se non c'è: il trigger non si attiva → verifica l'URL della thank-you page (contiene "thank-you"?), oppure l'evento custom non viene pushato nel data layer.

**Step 3 — Il tag si spara?**
- Clicca sul trigger → vedi i tag che si attivano?
- Se il trigger c'è ma il tag non si spara: controlla le condizioni di firing, eventuali eccezioni, e se il tag è in pausa.

**Step 4 — Le variabili sono popolate?**
- Nell'evento, vai su "Variables" → controlla che `DLV - transaction_id`, `DLV - ecommerce currency`, `Ecommerce Value` ecc. abbiano un valore.
- Se sono vuote: il data layer non viene pushato correttamente dallo sviluppatore.

**Step 5 — Il consenso blocca i tag?**
- Controlla se `GCM - Default Consent` si è attivato prima degli altri tag.
- Se usi Iubenda: testa accettando tutti i consensi e riverifica.

**Step 6 — Controlla nelle piattaforme di destinazione**
- GA4: DebugView (Analisi > DebugView) — vedi gli eventi in real-time?
- Meta: Events Manager > Test Events
- Google Ads: Tag Assistant, oppure verifica nella colonna "Conversioni" con un ritardo di 3-4 ore

---

### Problemi più comuni e soluzioni rapide

| Problema | Causa più probabile | Soluzione |
|---|---|---|
| Purchase non tracciata | URL thank-you non contiene "thank-you" | Cambiare URL o creare trigger ad hoc |
| Purchase non tracciata | Data layer `purchase` non pushato | Chiedere allo sviluppatore di verificare il plugin/integrazione |
| Tag non si spara | Consenso non accettato (GCM bloccante) | Verificare Iubenda, testare dopo consenso |
| Conversione duplicata | Trigger attivo su più pagine | Restringere trigger con condizioni aggiuntive |
| Pixel Meta senza valore | `DLV - ecommerce value` vuota | Verificare il data layer con lo sviluppatore |
| GA4 non vede eventi | Container in modalità bozza | Pubblicare la versione GTM |
| GADS conversione a 0 | Valore non passato al tag | Controllare la variabile `Conversion Value` nel tag |

---

## 3. ESCALATION — Report strutturato per Davide

Quando il problema non si risolve con la checklist sopra, raccogli queste informazioni prima di chiedere aiuto a Davide. Questo permette di intervenire senza tornare a fare le stesse domande.

Chiedi al collega (o raccogli tu stesso dal debug) e compila questo schema:

```
=== REPORT PROBLEMA TRACKING ===

📍 DOVE
- URL del sito: 
- Pagina specifica con il problema: 
- Container GTM ID (es. GTM-XXXXXX): 
- Piattaforma: [ ] WooCommerce  [ ] Shopify  [ ] Magento  [ ] Shopware  [ ] Altro: ___

🎯 COSA NON FUNZIONA
- Descrizione del problema: 
- Piattaforme coinvolte: [ ] GA4  [ ] Meta  [ ] Google Ads  [ ] TikTok  [ ] Altro: ___
- L'evento specifico che non si traccia: 
- Funzionava prima? [ ] Sì, da quando smesso: ___  [ ] No, non ha mai funzionato

📅 QUANDO
- Data/ora in cui è stato rilevato il problema: 
- È sempre riproducibile o intermittente? [ ] Sempre  [ ] Intermittente (frequenza: ___)
- Modifiche recenti al sito o al GTM: 

🔍 DEBUG GIÀ FATTO
- GTM Preview attivato: [ ] Sì  [ ] No
- Il container GTM si carica? [ ] Sì  [ ] No  [ ] Non so
- Il trigger si attiva? [ ] Sì  [ ] No  [ ] Non so — trigger controllato: ___
- Il tag si spara? [ ] Sì  [ ] No  [ ] Non so — tag controllato: ___
- Le variabili DLV sono popolate? [ ] Sì  [ ] No  [ ] Non so — variabili controllate: ___
- Testato dopo aver accettato i consensi Iubenda? [ ] Sì  [ ] No
- Screenshot/video del debug panel: [ ] Allegati  [ ] Non disponibili

📸 EVIDENZE
- Link screenshot GTM Preview: 
- Link al sito (se pubblico): 
- Note aggiuntive:
```

Una volta compilato, condividi il report nel canale team o direttamente con Davide.

---

## Note finali

- Quando lavori su un container di un cliente, **lavora sempre su una bozza** (workspace) — non pubblicare direttamente in produzione.
- Prima di pubblicare, aggiungi una nota alla versione (es. "Aggiunto tag Meta - generate_lead su click CTA header").
- Se non sei sicuro su come chiamare un tag o una variabile, controlla i template come riferimento.
- Per i template server-side: il container client e server devono avere gli stessi ID pixel.

Per approfondimenti:
- Naming completo → `references/naming-conventions.md`
- Setup per piattaforma → `references/platforms.md`
- Consent Mode avanzato → `references/consent-mode.md`
- Protocollo debug esteso → `references/debug-protocol.md`
