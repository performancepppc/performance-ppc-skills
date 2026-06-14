# Naming Conventions — GTM Performance PPC

Convenzioni di nomenclatura estratte dai container reali. Rispettarle mantiene i container
leggibili e coerenti tra tutti i clienti.

---

## TAG

### Struttura generale
`[PIATTAFORMA] - [tipo_evento_o_descrizione]`

### GA4
```
GA4 - Pageview
GA4 - G-XXXXXXXXXX          ← Google Tag (configurazione base)
GA4 - purchase
GA4 - begin_checkout
GA4 - view_item
GA4 - view_item_list
GA4 - add_to_cart
GA4 - remove_from_cart
GA4 - view_cart
GA4 - add_payment_info
GA4 - add_shipping_info
GA4 - generate_lead
GA4 - form_submit
GA4 - login
GA4 - sign_up
GA4 - internal_search
GA4 - search_results
GA4 - scroll_75
GA4 - view_30_sec
GA4 - exit_intent
GA4 - rage_click
GA4 - read_progress
GA4 - back_button
GA4 - click_tel
GA4 - click_mail
GA4 - click_whatsapp
GA4 - newsletter
GA4 - Event - Ecommerce events   ← tag aggregato per tutti gli eventi ecommerce
```

### Google Ads (GADS)
```
GADS - generate_lead
GADS - click telefono
GADS - click mail
GADS - click whatsapp
GADS - Scroll 75%
GADS - form_start
GADS - view_offer
GADS - Event - Ecommerce events
GADS - Dynamic Remarketing       ← tag remarketing dinamico
Linker - Google Ads              ← cross-domain linker
```

### Meta / Facebook Pixel
```
Meta - Pixel - [ID]             ← sitewide pixel
Meta - generate_lead
Meta - click telefono
Meta - click mail
Meta - click whatsapp
Meta - Scroll 75%
Meta - micro_conversion
META - [evento]                  ← variante uppercase nei template Elevar
Facebook - Sitewide Pixel        ← variante Elevar/vecchi template
Facebook - [evento]              ← variante Elevar
```

### Consent Mode (GCM)
```
GCM - Default Consent
GCM - Update consent
Consent Mode - Default           ← variante alternativa
Consent Mode - Update            ← variante alternativa
Consent Default                  ← variante semplificata
Consent Update                   ← variante semplificata
```

### Iubenda
```
Iubenda                          ← template Privacy Controls
Iubenda - Banner                 ← HTML banner (se non gestito dal template)
iubenda Privacy Controls and Cookie Solution   ← nome completo template
```

### Altre piattaforme
```
TikTok Pixel
TikTok - generate_lead
TikTok - Click telefono
TikTok - Click mail
TikTok - Scroll 75%
LinkedIn Insight Tag - Pixel
LinkedIn Insight Tag Pixel
Microsoft Clarity
Hotjar
CKS - [evento]                   ← Customer Key Solution (hashed user data)
```

### Utility / Custom HTML
```
cHtml - exit_intent
cHtml - view_30_sec
cHtml - form_error
cHtml - internal_search
cHtml - rage_click
cHtml - back_button
cHtml - read_progress
DL [evento]                      ← tag che pusha nel data layer
DL - Compilazione Form
DL - view_item
DL - add_to_cart
DL - purchase
```

---

## TRIGGER

### Pageview / Navigazione
```
Visualizzazione di pagina           ← All Pages (pageview)
Visualizzazione di pagina (No admin)← All Pages escluso admin/backend
Thank You - Generica               ← Page Path CONTAINS "thank-you"
Thank you ITA                      ← Page URL CONTAINS "[url-ita-thank-you]"
Thank you EN                       ← Page URL CONTAINS "[url-en-thank-you]"
Inizializzazione - All Pages       ← tipo: Consent Initialization
Inizializzazione - Consent accepted← dopo consenso Iubenda
Finestra Caricata                  ← Window Loaded
DOM Ready
```

### Ecommerce (Custom Events)
```
purchase                           ← customEvent "purchase"
begin_checkout                     ← customEvent "begin_checkout"
view_item                          ← customEvent "view_item"
view_item_list                     ← customEvent "view_item_list"
add_to_cart                        ← customEvent "add_to_cart"
remove_from_cart
view_cart
add_payment_info
add_shipping_info
analyzify_purchase                 ← specifico Shopify + Analyzify
```

### Elevar (Shopify)
```
Event - purchase
Event - begin_checkout
Event - user_data
Event - user_data - Thank You Page
Event - view_item
Event - view_item_list
Event - add_to_cart
Exception Event - Purchase Thank You Page
```

### Lead Generation / Engagement
```
generate_lead
form_submit
Form Submit
Form Start
generate_lead
newsletter
login
sign_up
Iscrizione Newsletter / Checkout   ← click su checkbox newsletter in checkout
```

### Interazioni utente
```
click_tel
click_mail
click_whatsapp
scroll_75 / Scroll 75
exit_intent
rage_click
view_30_sec
back_button
read_progress
internal_search
copyEmail
copyPhone
```

### Consent / GCM
```
GCM - Update consent
GCM - GTM update
Iubenda Consent Given
iubenda_consent_given_purpose_2
iubenda_consent_given_purpose_4 - Misurazione
iubenda_consent_given_purpose_5
iubenda_consent_update
Iubenda 1 - Necessary
Iubenda 2 - Basic functionalities
Iubenda 3 - Experience enhancement
Iubenda 4 - Measurement
Iubenda 5 - Targeting
```

---

## VARIABILI

### IDs (sempre costanti)
```
ID GA4                            ← G-XXXXXXXXXX
ID GTM                            ← GTM-XXXXXXX
ID Meta Pixel                     ← numero ID pixel
ID Conversione Google Ads         ← Customer ID GAds (es. 123456789)
ID TikTok Pixel
ID Clarity
ID LinkedIn
ID Hotjar
Pixel_ID                          ← alias per Meta Pixel (Elevar/Shopify)
GA4 ID                            ← alias per ID GA4 (alcuni template)
```

### Data Layer Variables (DLV)
Prefisso: `DLV - ` (uppercase con spazio trattino spazio)
```
DLV - Transaction ID
DLV - ecommerce currency
DLV - ECC - Order ID
DLV - ECC - Purchase revenue
DLV - search_term
DLV - search_results_url
DLV - error_message
DLV - element_label
DLV - read_percent
DLV - page_location
DLV - form_id
DLV - form_class
DLV - field_name
DLV - click_classes
DLV - click_text
DLV - click_url
DLV - Video Name
DLV - Video Action
DLV - User ID
DLV - User type
DLV - User Lifetime Value
DLV - User Existing Customer
DLV - visitor Type
DLV - Cookie banner status
DLV - Page type
DLV - Country Code
DLV - Store
DLV - Coupon
DLV - Scroll depth
```

Dati hashed ordine (WooCommerce avanzato):
```
DLV orderData.customer.billing.emailhash
DLV orderData.customer.billing.phone_hash
DLV orderData.customer.billing.first_name_hash
DLV orderData.customer.billing.last_name_hash
DLV orderData.customer.billing.email
DLV orderData.customer.billing.phone
DLV orderData.customer.billing.first_name
DLV orderData.customer.billing.last_name
DLV visitorEmailHash
DLV visitorType
```

### Variabili Ecommerce
```
Ecommerce Transaction ID
Ecommerce Value
Ecommerce Currency
Ecommerce Tax
Ecommerce Shipping
Ecommerce Coupon
Ecommerce Items
```

### Variabili Google Ads
```
Gads - Etichette conversione       ← lookup table etichette conversione
Gads - Dati forniti dagli utenti   ← user-provided data per enhanced conversions
GAds Dynamic Remarketing Items
GAds Dynamic Remarketing Value
Dati forniti dagli utenti          ← oggetto dati utente hashed
Dati forniti dagli utenti - Hashed
```

### Variabili Meta
```
Meta - contents (Add to Cart)
Meta - contents (view_content)
Meta - contents (checkout)
Meta Placement Value
js - META - Purchase - value
js - META - AddToCart - content_ids
```

### Variabili JavaScript / Custom
```
js - GA4 - view cart              ← custom JS variable
JS - Scroll Percent
JS - Click Y position
CJS - page path clean
CJSV - Country
CJSV - Language
CJSV - Custom task
```

### Consent / Iubenda
```
Consent Iubenda - 1 (Necessari)
Consent Iubenda - 2 (Funzionalità base)
Consent Iubenda - 3 (Miglioramento esperienza)
Consent Iubenda - 4 (Misurazione)
Consent Iubenda - 5 (Pubblicità)
Consent Iubenda - 5 (Pubblicità) true/false
LK - ad_storage
LK - analytics_storage
LK - personalization
cookie.purpose 1 - Necessary
cookie.purpose 2 - Basic functionalities
cookie.purpose 3 - Experience enhancement
cookie.purpose 4 - Measurement
cookie.purpose 5 - Targeting
iubenda siteId
iubenda cookiePolicyId
Iubenda - Cookie ID
Iubenda - Site ID
0_ cookie iubenda
0_ iubenda cookiePolicyId
```

### Variabili DPP / Customer Key (dati 1a parte)
```
DPP - formname
DPP - formsurname
DPP - formemail
DPP - formphone
Cks - Name
Cks - LastName
Cks - Mail
Cks - Phone
```

### Costanti e utili
```
constant - product group - product_group
constant - conversion value - subtotal
constant - product identifier - product id
timestamp
Unique Event ID
GA4 Event Settings                ← variabile impostazioni evento GA4
gclid_gtm                         ← GCLID per cross-domain
```
