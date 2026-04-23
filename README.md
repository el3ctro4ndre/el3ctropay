# el3ctropay
Are you looking for a fast, reliable and easy to use alternative to stripe?  
el3ctropay is what you need, a simple Solana payment gateway that can be used in any application via API. 

### Features and work in progress
- [x] API base system
- [x] API keys system
- [x] QR code and direct link payment 
- [x] Base UI style
- [x] Redirect back to original website once the payment is completed
- [x] Server callback to confirm the seller that the payment has been completed
- [x] UI Restyle

Everybody is welcome to open [issues](https://github.com/el3ctro4ndre/el3ctropay/issues) to suggest new features

### UI Restyling
| Before | After |
|--------|-------|
| ![](https://github.com/user-attachments/assets/5b530ebc-a99c-4f2a-a8b9-1290b9607b2a) | ![](https://github.com/user-attachments/assets/ceec403c-bb14-41d9-a10d-5bf6d1838416) |

### Disclaimer
el3ctropay makes the customer pay 0.0002 SOL more as a "processing fee", this cost equals about 0,015€, making the project able to sustain his own service costs to keep it online. (Euro value updated at 22th April 2026)

If you have any doubts you can check the project [Privacy Policy](https://pay.el3ctroservices.it/privacy-policy/) and the [Terms and Conditions](https://pay.el3ctroservices.it/terms-and-conditions/).

## DOCS

### Get an API Key
The authentication via API Key protects the payment gateway from spam and ensures privacy.  
Requesting an API Key is free of charge and can be done by contacting [@el3ctro4ndre](https://discord.com/users/617325296932356126) on discord.  
Once your request will be completed you will recieve both the API Key and a secret that you'll need in case you want to setup a webhook for automatic payment confirmation.  

Down below you can find a demo API key and a demo secret, any orders made with this API key are processed on the Solana devnet.  
API key: sk_live_ec4601d2fa78156f3df132a286449cb377d12408c351a645aa429bbe00e55def  
Secret: 54b3f8b4ef5a0bbc815ef1140b421062522e421f00eba52060198d5ba1e29e77

### Create a payment order
A payment order can be created via an authenticated API call, the code down below shows a python example using the requests module:
```
import requests

API_KEY = "YOUR_API_KEY"

r = requests.post("https://api.el3ctroservices.it/el3ctropay/create-order/", json={
    "label": "Acme Corp",
    "message": "Order N* 12345",
    "wallet": "WALLET_HERE", 
    "amount": 0.529,
    "redirect": "https://example.com/order-status?order=12345",
    "webhook": "https://example.com/api/webhook/"  -- Optional, use it in case you want automatic payment confirmation
}, headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
Once the payment order will be successfully creted you will receive a JSON response with this structure:
```
{ "payment_url": "https://pay.el3ctroservices.it/pay/?order=ORDER_ID" }
```

The only thing left to do is redirect the client to that URL.

### Fetch your order list
Fetching a complete order list and their respective payment status can be done via an authenticated API call, the code down below shows a python example using the requests module:
```
import requests

API_KEY = "YOUR_API_KEY"

r = requests.get("https://api.el3ctroservices.it/el3ctropay/fetch-orders/", headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
Onche the fetching will be succesfully created you will receive a JSON response with this structure:
```
{
    "orders": [
        {
            "reference": "ORDER_ID",
            "wallet": "WALLET_HERE",
            "paid": 1,                                        -- or 0 if the client hasn't paid yed
            "amount": 0.529,
            "link": "https://solscan.io/tx/TRANSACTION"       -- or null id the client hasn't paid yet
        },
        {
            ...
        }
    ]
```

### Automatic payment confirmation
To make your platform receive an automatic payment confirmation you can create an api endpoint on your site.  
All the confirmations are signed and you can verify them only by using your secret.  
The python code down below is an example on how to create a very simple FastAPI endpoint that can verify the message signature and print the transaction datas, you can put your own code to manage the received data but you must keep the `{"status": "ok"}` response to the gateway.
```
from fastapi import FastAPI, Request, Header, HTTPException
import hmac
import hashlib

SECRET = "YOUR_SECRET" 

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

    ### START OF THE DATA MANAGEMENT OF YOUR WEBSITE
    print("Payment received:")
    print(data)
    ### END OF THE DATA MANAGEMENT OF YOUR WEBSITE
    
    return {"status": "ok"}
```
The datas that the gateway will send in the automatic payment confirmation have the following JSON structure:
```
{
    "event": "transaction.paid"
    "amount": "0.529", 
    "reference": "ORDER_ID", 
    "tx": "TRANSACTION_CODE_OF_THE_SOLANA_BLOCKCHAIN"
}
```
