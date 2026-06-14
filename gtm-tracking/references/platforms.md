# Setup per Piattaforma — GTM Performance PPC

---

## WooCommerce (Base)
**Template:** `01_Template_EcommerceWoocommerce`

### Data layer
WooCommerce genera automaticamente eventi GA4 nel data layer se è installato il plugin
"Google Listings & Ads" o un plugin GA4 dedicato (es. DualSun, GTM4WP con ecommerce tracking).

Eventi principali nel DL:
- `purchase` → all'ordine confermato
- `begin_checkout` → all'inizio del checkout
- `add_to_cart`, `view_item`, `view_item_list`

### Thank you page
- URL default WooCommerce: `/checkout/order-received/[order-id]/`
- **Non contiene "thank-you"** → cambiare l'URL o creare trigger ad hoc:
  - Plugin "WooCommerce Custom Thank You" per cambiare l'URL
  - Oppure trigger: Page Path CONTAINS `order-received`
- Alternativa: usare il trigger `purchase` customEvent invece del pageview

### Pixel Meta
- Tag sitewide: `Meta - Pixel - [ID]`
- Evento purchase si basa sul customEvent `purchase` del data layer

### Nota
In WooCommerce il form newsletter al checkout usa `click_id = place_order` + checkbox newsletter.
Trigger: `Iscrizione Newsletter / Checkout` (tipo: click, ID CONTAINS "place_order" E checkbox checked)

---

## WooCommerce Avanzato
**Template:** `04_Template_Wocommerce_avanzato`

Versione avanzata con:
- **GA4 Server-Side** (container client + server)
- **Meta Pixel** via template custom (non snippet HTML)
- **GADS Enhanced Conversions** (dati utente hashed)
- **Dynamic Remarketing** Google Ads
- **Iubenda Consent Mode v2**

### Dati hashed richiesti nel data layer
Lo sviluppatore deve pushare nel DL all'acquisto:
```javascript
dataLayer.push({
  'orderData': {
    'customer': {
      'billing': {
        'emailhash': 'sha256_hash_email',
        'phone_hash': 'sha256_hash_phone',
        'first_name_hash': 'sha256_hash_nome',
        'last_name_hash': 'sha256_hash_cognome',
        'email': 'email_plain',
        'phone': 'telefono_plain',
        'first_name': 'nome_plain',
        'last_name': 'cognome_plain'
      }
    }
  }
});
```

### Thank you page
- Trigger `Thank you ITA` + `Thank you EN` per siti multi-lingua
- Trigger `Thank You - Generica` per siti mono-lingua
- Trigger `purchase` customEvent per il server-side

---

## Shopify con Analyzify
**Template:** `04_Template_EcommerceShopify`

### Data layer
Analyzify è un'app Shopify che genera automaticamente il data layer GA4.
L'evento di acquisto viene pushato come `analyzify_purchase` (oltre al classico `purchase`).

### Trigger principali
```
analyzify_purchase    ← per Meta Pixel e GADS (più affidabile)
purchase              ← alternativo, GA4
Thank You - Generica  ← pageview purchase (backup)
begin_checkout
begin_checkout / rapido   ← per il checkout rapido (buy now)
```

### Thank you page
- Su Shopify la thank you page è `/thank_you` (con underscore, non trattino!)
- Il trigger `Thank You - Generica` usa CONTAINS `thank-you` → **potrebbe non funzionare**
- Soluzione: aggiungere condizione CONTAINS `thank_you` OPPURE affidarsi ad `analyzify_purchase`

### Meta Pixel
- ID in variabile `ID Meta Pixel` oppure `Pixel_ID`
- Preferire `analyzify_purchase` come trigger per Purchase su Meta (più dati del DL)

---

## Shopify con Elevar
**Template:** `06_Template_Shopify Elevar` (avanzato) / `07 Template Elevar (default)` (base)

### Data layer
Elevar è un'app Shopify che genera un data layer con eventi prefisso `dl_`:
- `dl_purchase` → acquisto
- `dl_begin_checkout`
- `dl_view_item`
- `dl_view_item_list`
- `dl_add_to_cart`
- `dl_user_data` → dati utente per enhanced matching
- `dl_user_data_fired` → thank you page data (solo Elevar avanzato)

### Trigger
Nel template Elevar i trigger si chiamano:
```
Event - purchase
Event - begin_checkout
Event - user_data - Thank You Page
Event - view_item
Exception Event - Purchase Thank You Page
```

### Meta Pixel (Elevar)
I tag Facebook/Meta in Elevar usano HTML custom (`Facebook - [evento]`) con
advanced matching integrato dai dati del DL Elevar.
La variabile pixel si chiama `Facebook - Pixel ID` o `META - Pixel ID`.

### Note
- Elevar avanzato include già CAPI (Conversions API) per Meta
- Le variabili DL usano il prefisso lowercase `dlv -` (non `DLV -`)
- Il `GA4 ID` viene salvato come variabile constante `GA4 ID`

---

## Shopify Checkout Custom
**Template:** `08_Template | Shopify Checkout custom`

Simile al template Shopify Analyzify ma con eventi checkout personalizzati.
Usato quando il checkout è modificato (Shopify Plus) o quando si usano
eventi custom specifici del cliente.

Trigger aggiuntivi rispetto al template base:
- Trigger custom per step specifici del checkout

---

## Magento
**Template:** `03_Template_EcommerceMagento`

### Data layer
Magento usa il data layer ECC (Enhanced Ecommerce Classic - UA style):
```javascript
dataLayer.push({
  'event': 'purchase',
  'ecommerce': {
    'purchase': {
      'actionField': { 'id': '...', 'revenue': '...' },
      'products': [...]
    }
  }
});
```

### Thank you page
- URL: `/checkout/onepage/success/`
- Trigger: `Purchase` (tipo: pageview, Page Path CONTAINS `/checkout/onepage/success/`)
- Oppure customEvent `purchase` se il tema lo pushda

### Trigger ECC specifici
```
CE - ECC - Purchase
CE - ECC - Checkout
CE - ECC - Add to cart
GA – EEC Events (addToCart|checkout)
```

### Variabili ECC
```
DLV - ECC - Order ID
DLV - ECC - Purchase revenue
DLV - ECC - Checkout step
DLV - ECC - Product name add to cart
DLV - ECC - Product name detail views
```

---

## Shopware
**Template:** `10_Template_Ecommerce_Shopware`

### Data layer
Shopware usa variabili `transactionX` nel data layer:
```
DLV - Transaction ID
DLV - transaction Shipping
DLV - transaction Tax
DLV - transaction Payment Type
DLV - transactionShippingMethod
```

### Consent Mode multi-paese
Shopware è spesso usato da clienti con siti in più lingue (IT + DE + EN).
Iubenda richiede un `cookiePolicyId` diverso per paese:
- Variabile lookup table `0_ iubenda cookiePolicyId` → mappa paese → ID policy
- Variabile `0_ cookie iubenda` → switch in base al Country Code

### Trigger specifici
```
Event Equals Checkout
confirm_order*          ← evento custom Shopware alla conferma ordine
purchase
begin_checkout
```

---

## Sito Istituzionale (No Ecommerce)
**Template:** `02_Template_Sito Base`

### Tracking standard
- GA4 Pageview + eventi di engagement
- Meta Pixel Sitewide
- GADS generate_lead
- Opzionale: TikTok Pixel, LinkedIn Insight Tag, Clarity, Hotjar

### Conversioni tracciate (no purchase)
- `generate_lead` → form contatto, richiesta preventivo
- `click_tel`, `click_mail`, `click_whatsapp`
- `Scroll 75%`, `view_30_sec`, `exit_intent`
- `newsletter`

### Micro-conversioni (engagement)
- `rage_click`, `back_button`, `read_progress`
- `internal_search`, `search_results`
- `exit_intent`

---

## Setup Server-Side

Quando è presente un container server-side:
- Il container **client** invia dati al server GTM (URL server di performance-ppc.com o cliente)
- Il container **server** riceve e reinvia a GA4, Meta CAPI, GADS

### Variabili IDs nel server container
```
ID GA4              ← stesso Measurement ID del client
ID Conversione Google Ads  ← stesso Customer ID
Pixel ID Meta       ← stesso Pixel ID del client (nome può variare)
```

### Trigger server
- `GA4 > Purchase` → tipo ALWAYS, filtra su Client Name = "GA4" E Event Name = "purchase"
- `FB_CONVERSIONS_API-[PIXEL_ID]-Server-Trigger` → tipo ALWAYS, regex su Event Name
