# Consent Mode v2 — Setup Iubenda

---

## Setup Standard (Sito Mono-lingua)

### 1. Tag: GCM - Default Consent
- Tipo: template custom Consent Mode (cvt_K8GSG o simile)
- Trigger: `Inizializzazione - All Pages` (tipo: Consent Initialization - All Pages)
- Configurazione: tutti i segnali a `denied`

Segnali da impostare a denied di default:
```
ad_storage: denied
analytics_storage: denied
ad_personalization: denied
ad_user_data: denied
functionality_storage: denied
personalization_storage: denied
security_storage: granted   ← di solito granted per necessari
```

### 2. Tag: GCM - Update consent
- Tipo: stesso template Consent Mode
- Trigger: `iubenda_consent_update` oppure `Iubenda Consent Given`
- Configurazione: legge i consensi dall'evento Iubenda e aggiorna i segnali GCM

Mapping consensi Iubenda → GCM:
```
Iubenda Purpose 1 (Necessari)             → security_storage: granted
Iubenda Purpose 2 (Funzionalità base)     → functionality_storage
Iubenda Purpose 3 (Miglioramento esp.)    → personalization_storage
Iubenda Purpose 4 (Misurazione)           → analytics_storage
Iubenda Purpose 5 (Pubblicità/Targeting)  → ad_storage, ad_personalization, ad_user_data
```

### 3. Tag: Iubenda (template)
- Tipo: template "iubenda Privacy Controls and Cookie Solution"
- Trigger: `Inizializzazione - All Pages`
- Variabili richieste:
  - `iubenda siteId` → ID del sito su iubenda.com
  - `iubenda cookiePolicyId` → ID della cookie policy su iubenda.com

### 4. Tag: Iubenda - Banner (opzionale HTML)
- Solo se il template Iubenda non mostra il banner automaticamente
- Tipo: Custom HTML
- Contiene lo snippet `<script src="//cdn.iubenda.com/cs/iubenda_cs.js">` con configurazione

---

## Variabili per Iubenda

### Variabili di configurazione
```
iubenda siteId          → costante con il Site ID del cliente su iubenda.com
iubenda cookiePolicyId  → costante con il Privacy Policy/Cookie Policy ID
```

### Variabili di lettura consenso (per il tag Update)
```
Consent Iubenda - 1 (Necessari)
Consent Iubenda - 2 (Funzionalità base)
Consent Iubenda - 3 (Miglioramento esperienza)
Consent Iubenda - 4 (Misurazione)
Consent Iubenda - 5 (Pubblicità)
```

### Variabili lookup (per il mapping)
```
LK - ad_storage          → lookup: Purpose 5 accettato → "granted" / rifiutato → "denied"
LK - analytics_storage   → lookup: Purpose 4
LK - personalization     → lookup: Purpose 3
```

### Variabili cookie raw
```
cookie.purpose 1 - Necessary
cookie.purpose 2 - Basic functionalities
cookie.purpose 3 - Experience enhancement
cookie.purpose 4 - Measurement
cookie.purpose 5 - Targeting
```

---

## Setup Multi-paese / Multi-lingua (Shopware, WooCommerce multi-sito)

Quando il cliente ha più paesi/lingue con cookie policy diverse su Iubenda:

### Variabile lookup cookiePolicyId
Crea una variabile lookup table `0_ iubenda cookiePolicyId`:
```
Key (Country Code)  →  Value (cookiePolicyId)
IT                  →  [ID policy italiana]
DE                  →  [ID policy tedesca]
EN                  →  [ID policy inglese]
...
```

### Variabile cookiePolicyId dinamica
`0_ cookie iubenda` = variabile che restituisce il cookiePolicyId corretto in base
alla variabile `DLV - Country Code` (che a sua volta legge il country dal DL o URL).

### Trigger Iubenda per multi-lingua
Possono essere necessari trigger separati per consenso per lingua:
```
Iubenda Consent Given Purpose 4   ← Measurement per IT
iubenda_consent_given_purpose_4 - Misurazione
```

---

## Trigger per Consent Mode

### Trigger di inizializzazione
```
Inizializzazione - All Pages
- Tipo: Consent Initialization - All Pages
- Usato per: GCM Default, Iubenda template
```

### Trigger di aggiornamento consenso
```
iubenda_consent_update
- Tipo: Custom Event
- Nome evento: iubenda_consent_update
- Usato per: GCM Update

Iubenda Consent Given
- Tipo: Custom Event
- Nome evento: iubenda_consent_given
- Variante alternativa

GCM - GTM update
- Tipo: Window Loaded
- Usato per aggiornamento al caricamento della finestra
```

### Trigger per propósiti specifici
```
iubenda_consent_given_purpose_4 - Misurazione
iubenda_consent_given_purpose_5
Iubenda 1 - Necessary
Iubenda 4 - Measurement
Iubenda 5 - Targeting
```

---

## Verifica del Setup

Per verificare che il Consent Mode funzioni correttamente:

1. Apri GTM Preview sul sito
2. Nell'evento di inizializzazione, controlla "Consent State" → deve mostrare tutto `denied`
3. Accetta i consensi nel banner Iubenda
4. Verifica che `iubenda_consent_update` o `iubenda_consent_given` appaia nel DL
5. Controlla che il tag `GCM - Update consent` si sia sparato
6. In Consent State: `analytics_storage` deve essere `granted` dopo aver accettato

### Errori comuni
- **GCM Default non si spara prima degli altri tag**: verifica che il trigger sia di tipo
  "Consent Initialization", non un normale pageview
- **Update non aggiorna**: verifica il mapping dei parametri nel template GCM
- **Banner non compare**: verifica che il tag Iubenda sia publishato e che il siteId/cookiePolicyId siano corretti
- **Tag GA4/Meta non si sparano dopo consenso**: verifica che abbiano "Require additional consent checks" configurato correttamente
