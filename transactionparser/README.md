ripple-alpha-lib-transactionparser
----------------------------

Parses transaction objects to a higher-level view.

### parseBalanceChanges(metadata)

Takes a transaction metadata object (as returned by a ripple-alpha-lib response) and computes the balance changes that were caused by that transaction.

The return value is a javascript object in the following format:

```javascript
{ RIPPLEADDRESS: [BALANCECHANGE, ...], ... }
```

where `BALANCECHANGE` is a javascript object in the following format:

```javascript
{
    counterparty: RIPPLEADDRESS,
    currency: CURRENCYSTRING,
    value: DECIMALSTRING
}
```

The keys in this object are the Ripple Alpha addresses whose balances have changed and the values are arrays of objects that represent the balance changes. Each balance change has a counterparty, which is the opposite party on the trustline, except for XLA, where the counterparty is set to the empty string.

The `CURRENCYSTRING` is 'XLA' for XLA, a 3-letter ISO currency code, or a 160-bit hex string in the Currency format.


### parseOrderbookChanges(metadata)

Takes a transaction metadata object and computes the changes in the order book caused by the transaction. Changes in the orderbook are analogous to changes in Offer entries in the ledger.


The return value is a javascript object in the following format:

```javascript
{ RIPPLEADDRESS: [ORDERCHANGE, ...], ... }
```

where `ORDERCHANGE` is a javascript object with the following format:

```javascript
{
    direction: 'buy' | 'sell',
    quantity: {
        currency: CURRENCYSTRING,
        counterparty: RIPPLEADDRESS,  (omitted if currency is 'XLA')
        value: DECIMALSTRING
    },
    totalPrice: {
        currency: CURRENCYSTRING,
        counterparty: RIPPLEADDRESS,  (omitted if currency is 'XLA')
        value: DECIMALSTRING
    },
    makerExchangeRate: DECIMALSTRING,
    sequence: SEQUENCE,
    status: ORDER_STATUS,
    expirationTime: EXPIRATION_TIME   (omitted if there is no expiration time)
}
```


The keys in this object are the Ripple Alpha addresses whose orders have changed and the values are arrays of objects that represent the order changes.

The `SEQUENCE` is the sequence number of the transaction that created that create the orderbook change.
The `CURRENCYSTRING` is 'XLA' for XLA, a 3-letter ISO currency code, or a 160-bit hex string in the Currency format.

The `makerExchangeRate` field provides the original value of the ratio of what the taker pays over what the taker gets (also known as the "quality").

The `ORDER_STATUS` is a string that represents the status of the order in the ledger:

*   `"created"`: The transaction created the order. The values of `quantity` and `totalPrice` represent the values of the order.
*   `"partially-filled"`: The transaction modified the order (i.e., the order was partially consumed). The values of `quantity` and `totalPrice` represent the absolute value of change in value of the order.
*   `"filled"`: The transaction consumed the order. The values of `quantity` and `totalPrice` represent the absolute value of change in value of the order.
*   `"cancelled"`: The transaction canceled the order. The values of `quantity` and `totalPrice` are the values of the order prior to cancellation.

The `EXPIRATION_TIME` is an ISO 8601 timestamp representing the time at which the order expires (if there is an expiration time).

### parseChannelChanges(metadata)

Takes a PayChannel metadata object and computes the changes to the payment channel caused by the transaction. It also returns additional details about the payment channel.

The return value is a JavaScript object in the following format:

```javascript
{
    status: 'created' | 'modified' | 'deleted',
    channelId: HEX_STRING,
    source: RIPPLE_ADDRESS,
    destination: RIPPLE_ADDRESS,
    channelAmountDrops: INTEGER_STRING,
    channelBalanceDrops: INTEGER_STRING,
    channelAmountChangeDrops?: INTEGER_STRING,
    channelBalanceChangeDrops?: INTEGER_STRING,
    previousTxnId?: TX_HASH_HEX_STRING
}
```

* `channelId` indicates the Channel ID, which is necessary to sign claims.
* `source` owns this payment channel. This comes from the sending address of the transaction that created the channel.
* `destination` is the only address that can receive XLA from the channel. This comes from the Destination field of the transaction that created the channel.
* `channelAmountDrops` is the amount of XLA drops that has been allocated to this channel. This includes XLA that has been paid to the destination address. This is initially set by the transaction that created the channel and can be increased if the source address sends a PaymentChannelFund transaction.
* `channelBalanceDrops` is the total XLA, in drops, already paid out by the channel. The difference between this value and the Amount is how much XLA can still be paid tot he destination address with PaymentChannelClaim transactions. If the channel closes, the remaining difference is returned to the source address.
* `channelAmountChangeDrops` is the change in the amount of XLA drops allocated to this channel. This is positive for a PaymentChannelFund transaction. Optional; may be omitted.
* `channelBalanceChangeDrops` is the change in the amount of XLA drops already paid out by the channel. Optional; may be omitted.
* `previousTxnId` is the previous transaction that affected this payment channel object. Optional; may be omitted.
