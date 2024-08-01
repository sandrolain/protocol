# Structured LIghtweight MEssage Protocol (SLiMe)

The purpose of the SLiMe (Structured Lightweight Message Protocol) is to provide a lightweight communication method for channels based on TCP or UDP.

The protocol primarily consists of three parts:

- a header and optional sequences that contain information about message interpretation
- a format for serializing the payload that allows for semi-structured data definition
- a crc for message validation or signature

## Message Format

<table>
  <thead>
    <tr>
      <th>octet</th>
      <th colspan="8">0</th>
      <th colspan="8">1</th>
    </tr>
    <tr>
      <th>bit</th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
      <th>12</th>
      <th>13</th>
      <th>14</th>
      <th>15</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>header</th>
      <td colspan="3">version</td>
      <td>CRC flag</td>
      <td colspan="4">message type</td>
      <td colspan="4">ID length</td>
      <td colspan="4">Schema length</td>
    </tr>
    <tr>
      <th rowspan="4">Message ID</th>
      <td colspan="16">0-2 bytes</td>
    </tr>
    <tr>
      <td colspan="16">2-4 bytes</td>
    </tr>
    <tr>
      <td colspan="16">4-6 bytes</td>
    </tr>
    <tr>
      <td colspan="16">6-8 bytes</td>
    </tr>
    <tr>
      <th rowspan="4">Message Schema</th>
      <td colspan="16">0-2 bytes</td>
    </tr>
    <tr>
      <td colspan="16">2-4 bytes</td>
    </tr>
    <tr>
      <td colspan="16">4-6 bytes</td>
    </tr>
    <tr>
      <td colspan="16">6-8 bytes</td>
    </tr>
    <tr>
      <th>Payload</th>
      <td colspan="16">
        0-N bytes
      </td>
    </tr>
    <tr>
      <th rowspan="2">CRC</th>
      <td colspan="16">
        0-2 bytes
      </td>
    </tr>
    <tr>
      <td colspan="16">
        2-4 bytes
      </td>
    </tr>
  </tbody>
</table>

### Header

#### Version

The bits 0-2 of the header (the first 3 bits of the first byte) indicate the version of the protocol.  
The possible values range from 0 to 7.

#### CRC Flag

The 3rd bit of the header indicates if there is a CRC at the end of the message:

- `0`: no CRC
- `1`: with CRC

#### Message Type

The message type is identified by a sequence of 4 bits.

The first bit indicates whether the message is a request or a response:

- `0`: request
- `1`: response

The next 3 bits indicate the type of request or response.

The values for request messages are:

- `0b0000`: GENERIC / UNDEFINED
- `0b0001`: GET
- `0b0010`: POST
- `0b0011`: PUT
- `0b0100`: DELETE
- `0b0101`: ???
- `0b0110`: ???
- `0b0111`: ???

The values for response messages are:

- `0b1000`: OK (With payload)
- `0b1001`: ACCEPTED (No payload present)
- `0b1010`: INVALID REQUEST
- `0b1011`: UNAUTHORIZED
- `0b1100`: FORBIDDEN
- `0b1101`: NOT FOUND
- `0b1110`: TIMEOUT
- `0b1111`: SERVER ERROR

#### ID Length

The bits 8-11 of the header (the first 4 bits of the second byte) indicate the length in bytes of the message ID.  
The possible values range from 0 to 8.

#### Schema Length

The bits 12-15 of the header (the last 4 bits of the second byte) indicate the length in bytes of the message schema.
The possible values range from 0 to 8.

### Message ID

L'ID del messaggio è una sequenza di byte che serve a identificare l'univocità del messaggio ed eventualmente ad identificare un messaggio di risposta al relativo messaggio di richiesta. La sequenza di byte può essere intepretata come valore numerico o come stringa.  
La lunghezza dell'ID è variabile tra 0 e 8 byte e dipende da quanto indicato nell'apposita sezione dell'header.

### Message Schema

Lo schema del messaggio è una sequenza di byte che serve a identificare il tipo di struttura dati ed il significato dei valori del payload. La sequenza di byte può essere intepretata come valore numerico o come stringa.  
La lunghezza dello schema è variabile tra 0 e 8 byte e dipende da quanto indicato nell'apposita sezione dell'header.

### Payload

The payload is composed of a sequence of parameters divided into key-value pairs.

The key has a size of 2 bytes, while the value has a variable size.

The first 4 bits of the key indicate the type of the value:

- `0b0000`: bool (1 byte, valori ammessi 0x00 e 0x01)
- `0b0001`: int8 (1 byte)
- `0b0010`: int16 (2 byte)
- `0b0011`: int32 (4 byte)
- `0b0100`: int64 (8 byte)
- `0b0101`: float (4 byte)
- `0b0110`: double (8 byte)
- `0b0111`: short binary (length specified by 1 byte)
- `0b1000`: medium binary (length specified by 2 byte)
- `0b1001`: long binary (length specified by 4 byte)
- `0b1010`: short text (length specified by 1 byte)
- `0b1011`: medium text (length specified by 2 byte)
- `0b1101`: long text (length specified by 4 byte)
- `0b1110`: array (type and length specified by 2 byte with same logic of Param ID)
- `0b1111`: map (length specified by 2 byte)

The following 12 bits of the key indicate the parameter ID, which can be an integer from 0 to 4095.

The value of the parameter is specified based on the type indicated in the key.

#### bool

The `bool` type admits two values:

- `0x00`: false
- `0x01`: true

#### int8

The `int8` type requires 1 byte and allows for a signed integer value ranging from -127 to 127.

#### int16

The `int16` type requires 2 bytes and allows for a signed integer value ranging from -32_767 to 32_767.

#### int32

The `int32` type requires 4 bytes and allows for a signed integer value ranging from -2_147_483_647 to 2_147_483_647.

#### int64

The `int64` type requires 8 bytes and allows for a signed integer value ranging from -9_223_372_036_854_775_807 to 9_223_372_036_854_775_807.

#### float

The `float` type requires 4 bytes and allows for a signed floating-point value.

#### double

The `double` type requires 8 bytes and allows for a signed floating-point value.

#### short binary

The `short binary` type is formed by a variable number of bytes.
The first byte indicates the length in bytes of the value (from 0 to 255), followed by the byte sequence.

#### medium binary

The `medium binary` type is formed by a variable number of bytes.
The first 2 bytes indicate the length in bytes of the value (from 0 to 65_535), followed by the byte sequence.

#### long binary

The `long binary` type is formed by a variable number of bytes.
The first 4 bytes indicate the length in bytes of the value (from 0 to 4_294_967_295), followed by the byte sequence.

#### short text

The `short text` type is formed by a variable number of bytes.
The first byte indicates the length in bytes of the value (from 0 to 255), followed by the byte sequence.
Structurally equivalent to `short binary`, the value should be interpreted as a UTF-8 string.

#### medium text

The `medium text` type is formed by a variable number of bytes.
The first 2 bytes indicate the length in bytes of the value (from 0 to 65_535), followed by the byte sequence.  
Structurally equivalent to `medium binary`, the value should be interpreted as a UTF-8 string.

#### long text

The `long text` type is formed by a variable number of bytes.
The first 4 bytes indicate the length in bytes of the value (from 0 to 4_294_967_295), followed by the byte sequence.
Structurally equivalent to `long binary`, the value should be interpreted as a UTF-8 string.

#### array

The type `array` is formed by a variable number of bytes.
The first 2 bytes indicate the type of the elements and the length of the array.
The format of the first 2 bytes is similar to that of the key of the payload parameters:

- The first 4 bits indicate the type of the elements in the array
- The following 12 bits indicate the number of elements in the array (from 0 to 4095)

#### map

The type `map` is formed by a variable number of bytes.  
The first 2 bytes indicate the length of the map (from 0 to 65_535).  
The subsequent bytes are a sequence of parameters, equivalent in number to the length indicated in the first 2 bytes, which represents the key-value pairs of the map.  
The parameter sequence of the map is structurally equivalent to the main payload; it may therefore contain sub-structures of type `array` and `map`.

### CRC

The CRC is a sequence of 4 bytes (CRC-32) placed at the end of the message.
The presence of the CRC is optional and is indicated in the corresponding bit of the header.

## Message Schema Description

The message schema is an optional identifier that is useful for validating and identifying the message parameters.

----

- 2 byte: protocl header
  - 3 bit version
  - 1 bit crc (0=nocrc, 1=crc32)
  - 4 bit type
    - 0b0xxx: Request
      - 0b0000: GENERIC / UNDEFINED
      - 0b0001: GET
      - 0b0010: POST
      - 0b0011: PUT
      - 0b0100: DELETE
      - 0b0101: ???
      - 0b0110: ???
      - 0b0111: ???
    - 0b1000: Response
      - 0b1000: OK (previsto payload)
      - 0b1001: ACCEPTED (non deve essere presente payload)
      - 0b1010: INVALID REQUEST
      - 0b1011: UNAUTHORIZED
      - 0b1100: FORBIDDEN
      - 0b1101: NOT FOUND
      - 0b1110: TIMEOUT
      - 0b1111: SERVER ERROR
  - 4 bit ID length (valori validi nel range da 0-8)
  - 4 bit schema length (valori validi nel range da 0-8)
- (0-8) byte ID
- (0-8) byte schema
- N byte payload
  - 2 byte Param ID
    - 4 bit type
      - 0b0000: bool (1 byte, valori ammessi 0x00 e 0x01)
      - 0b0001: int8 (1 byte)
      - 0b0010: int16 (2 byte)
      - 0b0011: int32 (4 byte)
      - 0b0100: int64 (8 byte)
      - 0b0101: float (4 byte)
      - 0b0110: double (8 byte)
      - 0b0111: short binary (length specified by 1 byte)
      - 0b1000: medium binary (length specified by 2 byte)
      - 0b1001: long binary (length specified by 4 byte)
      - 0b1010: short text (length specified by 1 byte)
      - 0b1011: medium text (length specified by 2 byte)
      - 0b1101: long text (length specified by 4 byte)
      - 0b1110: array (type and length specified by 2 byte with same logic of Param ID)
      - 0b1111:map (length specified by 2 byte)
    - 12 bit ID: values from 0 to 4095
