## el3ctropay
Un semplice gateway di pagamento per solana

<img width="388" height="647" alt="image" src="https://github.com/user-attachments/assets/5b530ebc-a99c-4f2a-a8b9-1290b9607b2a" />

### ATTENZIONE
Al momento el3ctropay è in work in progress, quindi è attivo sulla solana devnet.

### Invia un suggerimento
Aprite degli [issues](https://github.com/el3ctro4ndre/el3ctropay/issues) per consigliarci metodi di server callback da implementare.

### Features e work in progress
- [x] Sistema API
- [x] Sistema Chiavi API
- [x] Pagamento con QR e Link 
- [x] Stile di base
- [x] Redirect al sito web originale una volta terminato il pagamento
- [ ] Stile avanzato (HTML e CSS proprietari con conseguente rimozione di bootstrap)
- [ ] Server callback per confermare il pagamento all'esercente

### Disclaimer
Quando si crea un ordine di pagamento riceverete esattamente la cifra inserita ma il cliente pagherà 0.0002 SOL in più come "tassa di processing", questo costo corrisponde a circca 0,015€, o un centesimo e mezzo di euro, permettendomi di mantenere attivo il servizio. 
(valore in euro aggiornato il 21 aprile 2026)

### Ottenere una chiave API
L'autenticazione tramite chiave API protegge il gateway di pagamento da spam, richiedere una chiave non comporta alcun costo.
È possibile richiedere una chiave API contattando @el3ctro4ndre su discord.

### Crea un ordine
Si può creare un ordine di pagamento su el3ctropay tramite una chiamata API, il seguente esempio mostra come inviare una chiamata API completa tramite la tua chiave:
```
API_KEY = "LA_TUA_CHIAVE_API"

r = requests.post("https://api.el3ctroservices.it/el3ctropay/create-order/", json={
    "label": "SOCIETÀ S.R.L.",
    "message": "NUMERO DI FATTURA, DI ORDINE O QUALSIASI MESSAGGIO A PIACIMENTO",
    "wallet": "IL WALLET DOVE RICEVERE I SOL", 
    "amount": LA QUANTITÀ DI SOL CHE DOVETE RICEVERE (esempio: 0.01)
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
API_KEY = "LA_TUA_CHIAVE_API"

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
}
```
