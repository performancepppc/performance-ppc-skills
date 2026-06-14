# Protocollo di Debug Esteso — GTM Performance PPC

---

## Strumenti da usare

### GTM Preview / Debug
1. Vai su https://tagmanager.google.com → apri il container → clicca "Preview"
2. Inserisci l'URL del sito e clicca "Connect"
3. Si apre il sito con il pannello debug in basso / in una finestra separata
4. Naviga sul sito e osserva gli eventi nel pannello

**Cosa guardare:**
- **Events (sinistra)**: lista di tutti gli eventi/trigger attivati
- **Tags (centro)**: per ogni evento, quali tag si sono sparati (Fired) e quali no (Not Fired)
- **Variables (centro)**: valori di tutte le variabili in quel momento
- **Data Layer (centro)**: il contenuto del data layer in quel momento

### GA4 DebugView
- Apri GA4 → Admin → DebugView
- Gli eventi compaiono in real-time (con 5-10 secondi di ritardo)
- Filtra per il tuo device con il parametro `debug_mode: true` (attivo automaticamente in GTM Preview)

### Meta Events Manager — Test Events
- Vai su Meta Ads Manager → Events Manager → seleziona il pixel → Test Events
- Inserisci l'URL del sito e attiva il test
- Vedi gli eventi in real-time

### Google Tag Assistant
- Estensione Chrome: https://tagassistant.google.com
- Utile per verifica rapida di GA4 e GADS

---

## Albero decisionale di debug

```
PROBLEMA: [descrivi qui il problema]
        |
        ▼
1. Il container GTM si carica?
   → NO: snippet GTM mancante o errato nel sito → chiama lo sviluppatore
   → SÌ: continua ▼

2. L'evento atteso appare nel pannello eventi GTM?
   → NO per eventi custom (purchase, begin_checkout...):
        il data layer non viene pushato → sviluppatore deve verificare plugin/integrazione
   → NO per pageview thank-you:
        verifica l'URL della pagina → contiene "thank-you"? (vedi sezione Thank You Page)
   → SÌ: continua ▼

3. Il tag desiderato è in "Fired" per quell'evento?
   → NO (tag in "Not Fired"):
        - Il trigger è corretto? Controlla le condizioni
        - Il tag è in pausa o ha eccezioni?
        - Il consenso lo sta bloccando? (vedi punto 5)
   → SÌ: continua ▼

4. Le variabili hanno i valori giusti?
   → NO (variabili vuote o undefined):
        - DLV non popolate: sviluppatore deve verificare il push nel DL
        - ID pixel vuoti: inserire il valore corretto nella variabile costante in GTM
   → SÌ: continua ▼

5. Il consenso blocca i tag?
   → In Consent State: analytics_storage o ad_storage sono "denied"?
        - Accetta tutti i consensi nel banner Iubenda
        - Riverifica → se ora funziona: il setup Consent Mode è corretto,
          il problema è l'utente che non accetta i consensi (normale)
        - Se non funziona ancora dopo consenso: il GCM Update non si sta sparando

6. Il dato arriva nella piattaforma?
   → GA4 DebugView: vedi l'evento? Con i parametri giusti?
   → Meta Test Events: vedi l'evento? Con valore e valuta?
   → GADS: controlla Conversion Actions, aspetta 3-4 ore per vedere i dati

7. Problema intermittente?
   → Controlla se dipende dal dispositivo (mobile vs desktop)
   → Controlla se dipende dal browser (Safari ha limitazioni sui cookie 3rd party)
   → Controlla se dipende dalla lingua/paese del sito
   → Verifica se c'è un A/B test attivo che modifica il checkout
```

---

## Scenari comuni con soluzione dettagliata

### Scenario 1: Purchase tracciata in GA4 ma non in Meta

**Causa più probabile**: il trigger `purchase` si attiva prima che Iubenda abbia aggiornato
il consenso `ad_storage`. Il tag GA4 ha `analytics_storage` già granted, ma Meta richiede
`ad_storage` che è ancora denied.

**Verifica**:
1. In GTM Preview, evento `purchase` → Tags
2. Il tag Meta è in "Not Fired" → guarda il motivo (di solito "Consent not granted")
3. Consent State all'evento purchase: `ad_storage` = ?

**Soluzione**:
- Verificare che il tag Meta abbia il requisito di consenso corretto impostato
- Oppure usare `analyzify_purchase` che si attiva dopo il caricamento della pagina
  (solitamente dopo l'accettazione del consenso)

---

### Scenario 2: Purchase tracciata in GA4 ma valore = 0 su GADS

**Causa più probabile**: la variabile `Conversion Value` nel tag GADS non è popolata.

**Verifica**:
1. In GTM Preview, evento `purchase` → Tag `GADS - Event - Ecommerce events`
2. Guarda il parametro "Conversion Value" → qual è il valore?
3. Controlla la variabile puntata (es. `DLV - ECC - Purchase revenue` o `Ecommerce Value`)

**Soluzione**:
- Verificare che la variabile DLV esista e sia configurata sul giusto path del data layer
- Nel DL `ecommerce.value` o `ecommerce.purchase.actionField.revenue` → dipende dalla piattaforma

---

### Scenario 3: Nessuna conversione in Google Ads, GA4 funziona

**Possibili cause**:
1. Il tag GADS non si spara (stesso scenario del tag GA4 ma con trigger diverso?)
2. La conversione in GADS non è importata da GA4 ma è un tag diretto → verifica il tag
3. Il parametro `Conversion Label` è sbagliato
4. Il `Conversion ID` (Customer ID) è sbagliato

**Verifica**:
1. GTM Preview → evento purchase → Tags → cerca il tag `GADS - ...`
2. Se "Fired": controlla i parametri nel tag (Conversion ID, Label, Value)
3. Se "Not Fired": torna all'albero decisionale punto 3

---

### Scenario 4: Il trigger thank-you page non si attiva

**Verifica**:
1. GTM Preview → naviga sulla pagina di conferma ordine
2. L'evento "Pageview" compare nel pannello?
3. Il trigger `Thank You - Generica` è in "Fired Triggers" o "Not Fired Triggers"?
4. Se "Not Fired": controlla l'URL della pagina

```
URL attuale:        https://example.com/ordine-confermato/
Trigger condizione: Page Path CONTAINS "thank-you"
Risultato:          NON MATCHA → il trigger non si attiva
```

**Soluzione**:
- Opzione A: chiedere allo sviluppatore di cambiare l'URL a `/thank-you/`
- Opzione B: creare un trigger ad hoc con la condizione corretta
  (es. Page Path CONTAINS `ordine-confermato` o l'URL specifico)
- Opzione C: usare il customEvent `purchase` invece del pageview come trigger principale

---

### Scenario 5: Conversioni duplicate in GADS/Meta

**Cause comuni**:
1. Il trigger si attiva più volte (es. pageview + customEvent entrambi attivi per lo stesso tag)
2. L'utente visita la thank-you page più volte (browser back button)
3. Il sito pushes l'evento purchase nel DL più di una volta

**Verifica**:
1. In GTM Preview, conta quante volte il tag si spara durante un acquisto
2. Controlla se ci sono più trigger associati allo stesso tag

**Soluzione**:
- Usare un trigger per ordine (es. solo `purchase` customEvent, non anche il pageview)
- Se usa pageview: aggiungere condizione aggiuntiva per assicurarsi che sia univoco
- Verificare con il parametro `transaction_id` che GADS/Meta deduplicano

---

### Scenario 6: Tags funzionano in Preview ma non in produzione

**Cause comuni**:
1. La versione GTM non è stata pubblicata → il Preview usa la bozza, il sito usa l'ultima versione pubblicata
2. Ci sono eccezioni basate sull'IP (non dovresti avere IP exceptions su GTM)
3. Il sito ha un sistema di caching che serve una versione vecchia del codice

**Verifica**:
1. Controlla la versione pubblicata in GTM (Versioni → vedi data ultima pubblicazione)
2. Pulisci la cache del sito (tramite il plugin/CMS o CDN) dopo aver pubblicato

---

## Template Report per Escalation a Davide

Da compilare quando i passaggi sopra non risolvono il problema:

```
=== REPORT PROBLEMA TRACKING ===

📍 DOVE
- URL sito: 
- Pagina specifica: 
- Container GTM ID: 
- Piattaforma: 

🎯 COSA NON FUNZIONA
- Descrizione: 
- Piattaforme coinvolte: 
- Evento specifico: 
- Funzionava prima? 

📅 QUANDO
- Rilevato il: 
- Riproducibile: [ ] Sempre  [ ] Intermittente
- Modifiche recenti: 

🔍 DEBUG GIÀ FATTO
- GTM Preview attivato: [ ] Sì  [ ] No
- Container si carica: [ ] Sì  [ ] No
- Trigger si attiva: [ ] Sì  [ ] No — trigger: ___
- Tag si spara: [ ] Sì  [ ] No — tag: ___
- Variabili DLV popolate: [ ] Sì  [ ] No — variabili: ___
- Testato dopo consenso Iubenda: [ ] Sì  [ ] No
- Pagine del DL viste: (copia qui il contenuto del DL all'evento)

📸 EVIDENZE
- Screenshot GTM Preview: 
- Link sito: 
- Note:
```
