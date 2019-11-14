# Nyx Checkout
> Payment service using credit card and bank slip.<br>
> To facilitate frontend integration of your project, our structure is socket-based.<br>


## Integration with Payment Services

* [Wirecard](https://wirecard.com.br/)

## How to use?

Access to the project services is possible via a socket connection and the transmission must provide a `socketID` attribute to identify the open channel and a` data` that will contain the data.<br>
To retrieve the return data you must listen to the socket from its `socketID`:
``` bash
socket.on('socketID', (data) => {})
```

## Services

### zipcode:fetch
CEP service that works integrated with the [VIACEP](https://viacep.com.br) to find out the information of the `zipcode` entered:

```js
socket.emit('zipcode:fetch', {
  socketID: 'abc',
  zipcode: '68900075'
})
```
---
### cities:fetch
Search service that retrieves a list of cities from Brazil:

```js
socket.emit('cities:fetch', {
  socketID: 'abc',
  data: {
    sigla_uf: 'AP'
  }
})
```
---
### freight:fetch
Freight calculation service:

```js
socket.emit('freight:fetch', {
  socketID: 'abc',
  data: {
    freight: {
      empresa: '',
      senha: '',
      maoPropria: 'N',
      avisoRecebimento: 'N',
      peso: '3.5',
      altura: '21',
      largura: '20',
      formato: '1',
      diametro: '1',
      comprimento: '20',
      valorDeclarado: 50,
      tipos: 'sedex,pac',
      cepOrigem: '52120310',
      cepDestino: '52111432'
    }
  }
})
```
---
### credit-card:encrypter
Credit card data encryption service:

```js
socket.emit('credit-card:encrypter', {
  socketID: 'abc',
  data: {
    creditCard: {
      holder: {
        fullname: 'xxx'
      },
      creditCardNumber: '5555666677778884',
      cvc: '123',
      expirationMonth: '06',
      expirationYear: '22'
    }
  }
})
```
---
### credit-card:post
Storage service that stores credit card data by linking it to a `userId`:

```js
socket.emit('credit-card:post', {
  socketID: 'abc',
  data: {
    creditCards: [{
      userId: 'xxx',
      name: 'xxx',
      hash: 'xxx',
      last4: 'xxxx',
      brand: 'xxx',
      address: {
        city: 'xxx',
        district: 'xxx',
        streetNumber: 'xxx',
        street: 'xxx',
        reference: 'xxx',
        state: 'xxx',
        zipCode: 'xxx'
      }
    }]
  }
})
```
---
### credit-card:fetch
Service that recovers saved credit card data from your `userId`:

```js
socket.emit('credit-card:fetch', {
  socketID: 'abc',
  data: {
    creditCard: {
      userId: 'xxx'
    }
  }
})
```
---
### credit-card:fetch:by-external-id
Service that retrieves credit card data for an externalId (ID used by Wirecard):

```js
socket.emit('credit-card:fetch:by-external-id', {
  socketID: 'abc',
  data: {
    creditCard: {
      externalId: 'xxx'
    }
  }
})
```
---
### checkout:create:
Service that creates a checkout:

```js
socket.emit('checkout:create', {
  socketID: 'abc'
})
```
Optionally a date object containing opening information can be sent to checkout:

```js
socket.emit('checkout:create', {
  socketID: 'abc',
  data: {
    user: {
      id: 'xxx',
      email: 'email@email'
    },
    products: [{
      id: '1',
      productName: 'xxx',
      detail: '<optional>',
      quantity: 100,
      price: 1032
    }]
  }
})
```
---
### checkout:close:
Service to close the checkout and send the data to the Wirecard, this method takes two forms of payment: `credit-card` and` bank-slip` that must be passed in the `method` parameter:

```js
socket.emit('checkout:close', {
  socketID: 'abc',
  data: {
    checkoutNumber: 'abc',
    holder: {
      fullname: 'xxx'
    },
    method: 'credit-card'
  }
})
```
---
### checkout:post:customer
Store service for credit card customer data:

```js
socket.emit('checkout:post:customer', {
  socketID: 'abc',
  data: {
    checkoutNumber: 'abc',
    user: {
      id: 'xxx',
      email: 'email@email'
    },
    customer: {
      kindPerson: 'xxx',
      name: {
        first: 'xxx',
        last: 'xxx'
      },
      fullname: 'xxx',
      email: 'email@email.com',
      birthDate: 'YYYY-MM-DD',
      taxDocument: {
        type: 'CPF|CNPJ',
        cnpj: 'xxx',
        cpf: 'xxx'
      },
      phone: {
        areaCode: '55',
        numberPhone: '<code><number>'
      },
      shippingAddress: {
        street: 'xxx',
        streetNumber: 'xxx',
        complement: 'xxx',
        district: 'xxx',
        city: 'xxx',
        state: 'xxx',
        zipCode: 'xxx'
      }
    }
  }
})
```
---
### checkout:post:products
Store service for product data to be checked out:

```js
socket.emit('checkout:post:products', {
  socketID: 'abc',
  data: {
    checkoutNumber,
    products: [{
      id: '1',
      name: 'xxx',
      quantity: 100,
      price: 1032
    }]
  }
})
```
---
### checkout:put:products
Update service that modifies information for a given product based on an `id`:

```js
socket.emit('checkout:put:products', {
  socketID: 'abc',
  data: {
    checkoutNumber,
    products: [{
      id: '1',
      name: 'yyy',
      quantity: 200,
      price: 2064
    }]
  }
})
```
---
## Webhook

We have resubmitted all notifications received from Wirecard. 
To do this you must register your hook using the http://your-project-checkout.nyx.tc/preferences 
route (POST) which receives the following parameters:

| Parametro | Descrição |
| --------  | -------- |
| media     | An array of strings for notification types. Receive: `PAYMENT` |
| events    | An array of strings containing the notification event types. All payments events `[CHECKOUT.CLOSED, CHECKOUT.CREATED, CHECKOUT.POST_CUSTOMER, CHECKOUT.POST_PRODUCTS, CHECKOUT.PUT_PRODUCTS]` and the [status for Wirecard PAYMENT events](https://dev.wirecard.com.br/reference#section-pagamentos) `[PAYMENT.CREATED, PAYMENT.WAITING, PAYMENT.IN_ANALYSIS, PAYMENT.PRE_AUTHORIZED, PAYMENT.AUTHORIZED, PAYMENT.CANCELLED, PAYMENT.REFUNDED, PAYMENT.REVERSED, PAYMENT.SETTLED]`|
| target    | Your notification receiving URL |
| tokenClient | String containing a client token|

#### Example

``` js
curl -X POST http://your-project-checkout.nyx.tc/preferences \
-d '{
      "media": ["PAYMENT"],
      "events": ["CHECKOUT_CREATED", "CHECKOUT_CLOSED"],
      "target": "http://your-project.com/checkout-notifications",
      "tokenClient": "<secret>"
    }'

# POST /preferences HTTP/1.1
# {
#     "media": [
#         "PAYMENT"
#     ],
#     "events": [
#         "CHECKOUT_CREATED",
#         "CHECKOUT_CLOSED"
#     ],
#     "target": "http://nameless.com/notify",
#     "token": "a50d-62bc070b5e6b-cc8a"
#     "id": "PR-DC53FD0A947"
# }
```

For [Wirecard](https://dev.wirecard.com.br/reference#section-pagamentos) events WIRECARD media and exclusive events should be used:

``` js
curl -X POST http://your-project-checkout.nyx.tc/preferences \
-d '{
      "media": ["PAYMENT"],
      "events": ["AUTHORIZED"],
      "target": "http://your-project.com/checkout-notifications",
      "tokenClient": "<secret>"
    }'

# POST /preferences HTTP/1.1
# {
#     "media": [
#         "PAYMENT"
#     ],
#     "events": [
#         "AUTHORIZED"
#     ],
#     "target": "http://nameless.com/notify",
#     "token": "a50d-62bc070b5e6b-cc8a"
#     "id": "PR-DC53FD0A947"
# }
```
---
### How to remove a webhook?

Registered hooks can be removed by the "id" provided during your registration, the return should be 204:

#### Example

``` js
curl -X POST http://your-project-checkout.nyx.tc/preferences/remove \
-d '{
      "id": "PR-DC53FD0A947",
      "tokenClient": "<secret>"
    }'

# POST /preferences/remove HTTP/1.1
# 204
```
---
### How to recover webhooks?

#### Example

``` js
curl -X GET http://your-project-checkout.nyx.tc/preferences/notifications

# GET /preferences/notifications HTTP/1.1
# {
#     "data": [{
#          "media": [
#              "PAYMENT"
#          ],
#          "events": [
#              "CHECKOUT_CREATED"
#              "CHECKOUT_CLOSED"
#              "AUTHORIZED"
#          ],
#          "target": "http://nameless.com/notify",
#          "token": "a50d-62bc070b5e6b-cc8a"
#          "id": "PR-DC53FD0A947"
#          "updatedAt": "2019-06-07T18:43:27.014Z"
#     }]
# }
```
