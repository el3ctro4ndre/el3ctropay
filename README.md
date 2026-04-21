## el3ctropay
A simple solana payment gateway

### GET A KEY
You can request an API key via email at mail@example.com

### CREATE AN ORDER
You can create a payment order on our gateway via our API, here there's a Python example on how to send a complete API call with your key: 
```
API_KEY = "YOUT_API_KEY"

r = requests.post("https://api.el3ctroservices.it/el3ctropay/create-order/", json={
    "label": "Società Inutili al Piripillo Srl",
    "message": "Ordine N° 21934",
    "wallet": "H9VGCU5aHKPhHu255GcGHEKgfZonQaMe7D4dBMe4kQQw", 
    "amount": 0.01
}, headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
If the order creation happened correctly you will receive a response like this one:
```
{ "payment_page": "https://pay.el3ctroservices.it/pay/?order=ORDER_ID" }
```

When you receive the URL you just need to redirect your client to that page.

### FETCH ALL YOUR TRANSACTIONS
To get a complete list of the transactions you created and their status you can use this simple call:
```
API_KEY = "YOUT_API_KEY"

r = requests.get("https://api.el3ctroservices.it/el3ctropay/fetch-orders/", headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
You will receive a response in JSON list like this:
```
{
    "orders": [
        {
            "reference": "UNIQUE EL3CTROPAY TRANSACTION ID",
            "wallet": "YOUR WALLET",
            "paid": 1,                                        -- or 0 if the order hasn't been paid yet
            "amount": YOUR AMOUNT,
            "link": "https://solscan.io/tx/TRANSACTION"       -- or null if the order hasn't been paid yet
        },
        {
            ...
        }
    ]
}
```
