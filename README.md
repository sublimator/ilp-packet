# ILP Packet

[![npm][npm-image]][npm-url] [![circle][circle-image]][circle-url] [![codecov][codecov-image]][codecov-url]

[npm-image]: https://img.shields.io/npm/v/ilp-packet.svg?style=flat
[npm-url]: https://npmjs.org/package/ilp-packet
[circle-image]: https://circleci.com/gh/interledgerjs/ilp-packet.svg?style=shield
[circle-url]: https://circleci.com/gh/interledgerjs/ilp-packet
[codecov-image]: https://codecov.io/gh/interledgerjs/ilp-packet/branch/master/graph/badge.svg
[codecov-url]: https://codecov.io/gh/interledgerjs/ilp-packet

> A serializer and deserializer for ILP packets and messages

## Usage

### Installation

```sh
npm install ilp-packet
```

### Deserialize ILP Packet

```js
const packet = require('ilp-packet')

const binaryPacket = Buffer.from('011c000000000754d4c00e672e75732e6e657875732e626f620304104100', 'hex')
const jsonPacket = packet.deserializeIlpPacket(binaryPacket)
console.log(jsonPacket)
// prints {
//   type: 1,
//   typeString: 'ilp_payment',
//   data: {
//     amount: '123000000',       // Unsigned 64-bit integer as a string
//     account: 'g.us.nexus.bob', // ILP Address
//     data: 'BBBB'               // Base64url-encoded attached data
//   }
// }
```

### Serialize/deserialize ILP Payment Packet

```js
const packet = require('ilp-packet')

const binaryPacket = packet.serializeIlpPayment({
  amount: '123000000',       // Unsigned 64-bit integer as a string
  account: 'g.us.nexus.bob', // ILP Address
  data: 'BBBB'               // Base64url-encoded attached data
}) // returns a Buffer

console.log(binaryPacket.toString('hex'))
// prints '011c000000000754d4c00e672e75732e6e657875732e626f620304104100'

const jsonPacket = packet.deserializeIlpPayment(binaryPacket)
```

### Serialize/deserialize ILQP Quote Requests/Responses

#### IlqpLiquidityRequest

```js
const packet = require('ilp-packet')

const binary = packet.serializeIlqpLiquidityRequest({
  destinationAccount: 'example.nexus.bob',
  destinationHoldDuration: 3000
})

const json = packet.deserializeIlqpLiquidityRequest(binary)
```

#### IlqpLiquidityResponse

```js
const packet = require('ilp-packet')

const binary = packet.serializeIlqpLiquidityResponse({
  liquidityCurve: Buffer.alloc(16), // Must be a buffer of size (n * 16) bytes
                                    // where n is the number of points in the
                                    // curve.
  appliesToPrefix: 'example.nexus.',
  sourceHoldDuration: 15000,
  expiresAt: new Date()
})

const json = packet.deserializeIlqpLiquidityResponse(binary)
```

### IlqpBySourceRequest

```js
const packet = require('ilp-packet')

const binary = packet.serializeIlqpBySourceRequest({
  destinationAccount: 'example.nexus.bob',
  sourceAmount: '9000000000',
  destinationHoldDuration: 3000
})

const json = packet.deserializeIlqpBySourceRequest(binary)
```

### IlqpBySourceResponse

```js
const packet = require('ilp-packet')

const binary = packet.serializeIlqpBySourceResponse({
  destinationAmount: '9000000000',
  sourceHoldDuration: 3000
})

const json = packet.deserializeIlqpBySourceResponse(binary)
```

### IlqpByDestinationRequest

```js
const packet = require('ilp-packet')

const binary = packet.serializeIlqpByDestinationRequest({
  destinationAccount: 'example.nexus.bob',
  destinationAmount: '9000000000',
  destinationHoldDuration: 3000
})

const json = packet.deserializeIlqpByDestinationRequest(binary)
```

### IlqpByDestinationResponse

```js
const packet = require('ilp-packet')

const binary = packet.serializeIlqpByDestinationResponse({
  sourceAmount: '9000000000',
  sourceHoldDuration: 3000
})

const json = packet.deserializeIlqpByDestinationResponse(binary)
```
### IlpError

```js
const packet = require('ilp-packet')

const binary = packet.serializeIlpError({
  code: 'F01',
  name: 'Invalid Packet',
  triggeredBy: 'example.us.ledger3.bob',
  forwardedBy: [
    'example.us.ledger2.connie',
    'example.us.ledger1.conrad'
  ],
  triggeredAt: new Date(),
  data: JSON.stringify({
    foo: 'bar'
  })
})

const json = packet.deserializeIlpError(binary)

const additionalErrorData = JSON.parse(json.data)
```

### IlpFulfillment

```js
const packet = require('ilp-packet')

const binary = packet.serializeIlpFulfillment({
  data: 'BBBB'               // Base64url-encoded attached data
})

const json = packet.deserializeIlpFulfillment(binary)
```

### IlpPrepare, IlpFulfill, IlpReject

```js
const packet = require('ilp-packet')
const crypto = require('crypto')
function sha256 (preimage) { return crypto.createHash('sha256').update(preimage).digest() }

const fulfillment = crypto.randomBytes(32)
const condition = sha256(fulfillment)

const binaryPrepare = packet.serializeIlpPrepare({
  amount: '10',
  executionCondition: condition,
  destination: 'g.us.nexus.bob', // this field was called 'account' in older packet types
  data: Buffer.from(['hello world']),
  expiresAt: new Date(new Date().getTime() + 10000)
})

const binaryFulfill = packet.serializeIlpFulfill({
  fulfillment,
  data: Buffer.from('thank you')
})

// not to be confused with IlpRejection:
const binaryReject = packet.serializeIlpReject({
  code: 'F00',
  triggeredBy: 'g.us.nexus.gateway',
  message: 'more details, human-readable',
  data: Buffer.from('more details, machine-readable')
})
```
