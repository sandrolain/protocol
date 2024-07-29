# {name} Protocol (W.I.P.)

L'obiettivo che si pone il protocollo {name} è di fornire uno metodo di comunicazione leggero per canali basati su TCP o UDP.

Il protocollo definisce principalmente tre parti:

- un header e alcune sequenze opzionali contenenti le informazioni sull'interpretazione del messaggio
- un formato per la serializzazione del payload che permetta di definire dati semi-strutturati
- un crc per la validazione e/o signature del messaggio

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
      <td rowspan="4">Message ID</td>
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
      <td rowspan="4">Message Schema</td>
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
      <td>Payload</td>
      <td colspan="16">
        0-N bytes
      </td>
  </tbody>
</table>

## Message Type

Il tipo di messaggio è identificato da una sequenza di 4 bit.

Il primo bit indica se il messaggio è una richiesta o una risposta:

- 0 = richiesta
- 1 = risposta

I successivi 3 bit indicato il tipo di richiesta o di risposta.

I valori per i messaggi di richiesta sono:

- 0b0000: GENERIC / UNDEFINED
- 0b0001: GET
- 0b0010: POST
- 0b0011: PUT
- 0b0100: DELETE
- 0b0101: ???
- 0b0110: ???
- 0b0111: ???

I valori per i messaggi di risposta sono:

- 0b1000: OK (previsto payload)
- 0b1001: ACCEPTED (non deve essere presente payload)
- 0b1010: INVALID REQUEST
- 0b1011: UNAUTHORIZED
- 0b1100: FORBIDDEN
- 0b1101: NOT FOUND
- 0b1110: TIMEOUT
- 0b1111: SERVER ERROR

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
