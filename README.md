# el3ctropay
A simple Solana payment gateway.

### WARNING
At the moment el3ctropay is still a work in progress, it's fully working on the solana devnet.

### Features and work in progress
- [x] API base system
- [x] API keys system
- [x] QR code and direct link payment 
- [x] Base UI style
- [x] Redirect back to original website once the payment is completed
- [x] Server callback to confirm the seller that the payment has been completed
- [x] UI Restyle
- [ ] Translate from Italian to English (I should have written everything in english at first)

Everybody is welcome to open [issues](https://github.com/el3ctro4ndre/el3ctropay/issues) to suggest new features

### UI Restyling (later image in english language coming soon
| Before | After |
|--------|-------|
| ![](https://github.com/user-attachments/assets/5b530ebc-a99c-4f2a-a8b9-1290b9607b2a) | ![](https://github.com/user-attachments/assets/08a0b9dc-5247-4e9f-8329-db92d9cca991) |

### Disclaimer
el3ctropay makes the customer pay 0.0002 SOL more as a "processing fee", this cost equals about 0,015€, also told one and a half euro cent, making the project able to sustain the service costs to keep it online. (Euro value updated at 22th April 2026)

## DOCS

### Get an API Key
The authentication via API Key protects the payment gateway from spam and ensuring privacy.
Requesting an API Key is free of charge and can be done by contacting [@el3ctro4ndre](https://discord.com/users/617325296932356126) on discord.
The API Key is sent to you with a secret that you'll need in case you want to setup a webhook on where to receive automatic payment confirmation.

### Create a payment order
A payment order can be created via an authenticated API call, the code down below shows a python example of how it can be done:
```
API_KEY = "YOUR_API_KEY"

r = requests.post("https://api.el3ctroservices.it/el3ctropay/create-order/", json={
    "label": "SOCIETÀ S.R.L.",
    "message": "NUMERO DI FATTURA, DI ORDINE O QUALSIASI MESSAGGIO A PIACIMENTO",
    "wallet": "IL WALLET DOVE RICEVERE I SOL", 
    "amount": LA QUANTITÀ DI SOL CHE DOVETE RICEVERE (esempio: 0.01),
    "redirect": "URL PAGINA DOVE REINDIRIZZARE IL CLIENTE A PAGAMENTO AVVENUTO",
    "webhook": "WEBHOOK DOVE RICEVERE LA CONFERMA DEL PAGAMENTO (https://example.com/webhook)"  -- OPZIONALE
}, headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
Se la creazione dell'ordine avviene con successo si riceverà una risposta con questo formato JSON:
```
{ "payment_url": "https://pay.el3ctroservices.it/pay/?order=ORDER_ID" }
```

Una volta ricevuto l'URL di pagamento si deve solamente reindirizzare a quella pagina il cliente.

### Ottenere la lista dei tuoi ordini
Per ottenere una lista completa degli ordini di pagamento che creati e visionare il loro stato si può usare questa semplice chiamata:
```
API_KEY = "LA_VOSTRA_CHIAVE_API"

r = requests.get("https://api.el3ctroservices.it/el3ctropay/fetch-orders/", headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
Si riceverà una risposta con questo formato JSON:
```
{
    "orders": [
        {
            "reference": "ID ORDINE UNICO",
            "wallet": "IL TUO WALLET",
            "paid": 1,                                        -- o 0 se l'ordine non è ancora stato pagato dal cliente
            "amount": QUANTITÀ,
            "link": "https://solscan.io/tx/TRANSACTION"       -- o null se l'ordine non è ancora stato pagato dal cliente
        },
        {
            ...
        }
    ]
```

### Ricevere la conferma del pagamento
Se si vuole ricevere una conferma automatizzata a pagamento avvenuto si può usare il seguente codice come esempio per creare un webhook in FastAPI da inoltrare al gateway per ricevere i dati di conferma, la richiesta inviata dal gateway è firmata e nel codice di esempio si ha anche la parte di verifica della firma tramite il vostro secret associato alla vostra chiave API.
```
from fastapi import FastAPI, Request, Header, HTTPException
import hmac
import hashlib

SECRET = "IL_VOSTRO_SECRET" 

app = FastAPI()

def verify_signature(body: bytes, signature: str, secret: str):
    expected = hmac.new(
        secret.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)

@app.post("/webhook/")
async def webhook(request: Request, x_signature: str = Header(None)):
    body = await request.body()

    if not verify_signature(body, x_signature, SECRET):
        raise HTTPException(status_code=400, detail="Invalid signature")

    data = await request.json()

    # Inizio del vostro codice per gestire i dati ricevuti
    print("Pagamento ricevuto:")
    print(data)
    # Fine del vostro codice per gestire i dati ricevuti
    
    return {"status": "ok"}
```
i dati che riceverete in automatico saranno tutti stringhe e avranno questo formato JSON:
```
{
    "amount": "LA QUANTITÀ DI SOL CHE AVETE RICEVUTO", 
    "reference": "ID ORDINE UNICO", 
    "tx": "CODICE TRANSAZIONE SULLA BLOCKCHAIN SOLANA"
}
```
