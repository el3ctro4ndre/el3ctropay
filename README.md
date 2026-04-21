## el3ctropay
Un semplice gateway di pagamento solana

### ATTENZIONE
Al momento el3ctropay è in work in progress, quindi è attivo sulla solana devnet.

### Ottenere una chiave API
Puoi richiedere una chiave API contattando @el3ctro4ndre su discord.

### Crea un ordine
Puoi creare un ordine di pagamento su el3ctropay tramite una chiamata API, il seguente esempio mostra come inviare una chiamata API completa tramite la tua chiave:
```
API_KEY = "LA_TUA_CHIAVE_API"

r = requests.post("https://api.el3ctroservices.it/el3ctropay/create-order/", json={
    "label": "SOCIETÀ S.R.L.",
    "message": "NUMERO DI FATTURA O QUALSIASI MESSAGGIO A TUO PIACIMENTO",
    "wallet": "IL WALLET DOVE RICEVERE I SOL", 
    "amount": LA QUANTITÀ DI SOL CHE DOVETE RICEVERE (esempio: 0.01)
}, headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
Se la creazione dell'ordine avviene con successo riceverai una risposta con questo formato JSON:
```
{ "payment_url": "https://pay.el3ctroservices.it/pay/?order=ORDER_ID" }
```

Ricevuto l'URL di pagamento dovrete solamente reindirizzare a quella pagina il cliente.

### Ottieni la lista dei tuoi ordini
Per ottenere una lista completa degli ordini di pagamento che hai creato e visionare il loro stato puoi usare questa semplice chiamata:
```
API_KEY = "LA_TUA_CHIAVE_API"

r = requests.get("https://api.el3ctroservices.it/el3ctropay/fetch-orders/", headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
Riceverai una risposta con questo formato JSON:
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
}
```
