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
