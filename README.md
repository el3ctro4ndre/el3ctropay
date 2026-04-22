# el3ctropay
Are you looking for a fast, easy and reliable alternative to stripe? el3ctropay is what you need, a simple solana payment gateway that can be used in any application via API. 

### WARNING
At the moment el3ctropay is still a work in progress, it's fully working on the solana devnet.

Since the gateway is still on the devnet I created a public API key and secret that people can use as demo and testing, since these keys are available to be used by everyone don't use them for final/personal use:  

API key: sk_live_ec4601d2fa78156f3df132a286449cb377d12408c351a645aa429bbe00e55def  
Secret: 54b3f8b4ef5a0bbc815ef1140b421062522e421f00eba52060198d5ba1e29e77

### Features and work in progress
- [x] API base system
- [x] API keys system
- [x] QR code and direct link payment 
- [x] Base UI style
- [x] Redirect back to original website once the payment is completed
- [x] Server callback to confirm the seller that the payment has been completed
- [x] UI Restyle
- [x] README.md translation from Italian to English (I should have written everything in english at first)
- [x] UI translation from Italian to English (I should have written everything in english at first)

Everybody is welcome to open [issues](https://github.com/el3ctro4ndre/el3ctropay/issues) to suggest new features

### UI Restyling
| Before | After |
|--------|-------|
| ![](https://github.com/user-attachments/assets/5b530ebc-a99c-4f2a-a8b9-1290b9607b2a) | ![](https://github.com/user-attachments/assets/08a0b9dc-5247-4e9f-8329-db92d9cca991) |

### Disclaimer
el3ctropay makes the customer pay 0.0002 SOL more as a "processing fee", this cost equals about 0,015€, making the project able to sustain his own service costs to keep it online. (Euro value updated at 22th April 2026)

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
    "label": "YOUR SOCIETY LLC",
    "message": "ORDER NUMBER OR ANY MESSAGE OF YOUR LIKING",
    "wallet": "WALLET WHERE YOU WANT TO RECEIVE THE SOL", 
    "amount": QUANTITY OF SOL YOU WANT TO RECEIVE (example: 0.529),
    "redirect": "URL OF THE PAGE THE CLIENT WILL BE REDIRECTED AFTER THE PAYMENT HAS SUCCEDED",
    "webhook": "WEBHOOK URL TO RECEIVE THE AUTOMATIC PAYMENT CONFIRMATION"  -- OPTIONAL
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
To fetch a complete order list with their status you can use this API call:
```
API_KEY = "YOUR_API_KEY"

r = requests.get("https://api.el3ctroservices.it/el3ctropay/fetch-orders/", headers={
    "Authorization": f"Bearer {API_KEY}"
})
```
You will receive a JSON response with this list structure:
```
{
    "orders": [
        {
            "reference": "UNIQUE ORDER ID",
            "wallet": "YOUR WALLET",
            "paid": 1,                                        -- or 0 if the client hasn't paid yed
            "amount": SOL QUANTITY,
            "link": "https://solscan.io/tx/TRANSACTION"       -- or null id the client hasn't paid yet
        },
        {
            ...
        }
    ]
```

### Automatic payment confirmation
To make your platform receive an automatic payment confirmation you can create an api endpoint the gateway will use. 
All the confirmations are signed and only with your secret you can verify them.
The python code down below is an example on how to create a very simple FastAPI endpoint that can verify the message signature, you can put your own code to manage the received data but you must keep the `{"status": "ok"}` response to the gateway.
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
    print("Pagamento ricevuto:")
    print(data)
    ### END OF THE DATA MANAGEMENT OF YOUR WEBSITE
    
    return {"status": "ok"}
```
The datas that the gateway will send in the automatic payment confirmation have the following JSON structure:
```
{
    "amount": "SOL QUANTITY YOU HAVE RECEIVED", 
    "reference": "UNIQUE ORDER ID", 
    "tx": "TRANSACTION CODE OF THE SOLANA BLOCKCHAIN"
}
```
