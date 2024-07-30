# Structured LIghtweight MEssage Protocol (SLiMe)

L'obiettivo che si pone il protocollo SLiMe è di fornire uno metodo di comunicazione leggero per canali basati su TCP o UDP.

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

## Header

### Version

I bit 0-2 bit dell'header (i primi 3 bit del primo byte) indicano la versione del protocollo. I valori possibili rientrano nel range 0-7.

### CRC Flag

Il bit 3 dell'header indica se è presente il CRC alla fine del messaggio.
Valore 0 indica assenza del CRC, valore 1 indica la presenza del CRC.

### Message Type

Il tipo di messaggio è identificato da una sequenza di 4 bit.

Il primo bit indica se il messaggio è una richiesta o una risposta:

- `0` = richiesta
- `1` = risposta

I successivi 3 bit indicato il tipo di richiesta o di risposta.

I valori per i messaggi di richiesta sono:

- `0b0000`: GENERIC / UNDEFINED
- `0b0001`: GET
- `0b0010`: POST
- `0b0011`: PUT
- `0b0100`: DELETE
- `0b0101`: ???
- `0b0110`: ???
- `0b0111`: ???

I valori per i messaggi di risposta sono:

- `0b1000`: OK (previsto payload)
- `0b1001`: ACCEPTED (non deve essere presente payload)
- `0b1010`: INVALID REQUEST
- `0b1011`: UNAUTHORIZED
- `0b1100`: FORBIDDEN
- `0b1101`: NOT FOUND
- `0b1110`: TIMEOUT
- `0b1111`: SERVER ERROR

### ID Length

I bit 8-11 dell'header (primi 4 bit del secondo byte) indicano la lunghezza in byte dell'ID del messaggio.  
I valori possibili rientrano nel range 0-8.

### Schema Length

I bit 12-15 dell'header (ultimi 4 bit del secondo byte) indicano la lunghezza in byte dello schema del messaggio.  
I valori possibili rientrano nel range 0-8.

## Message ID

L'ID del messaggio è una sequenza di byte che serve a identificare l'univocità del messaggio ed eventualmente ad identificare un messaggio di risposta al relativo messaggio di richiesta. La sequenza di byte può essere intepretata come valore numerico o come stringa.  
La lunghezza dell'ID è variabile tra 0 e 8 byte e dipende da quanto indicato nell'apposita sezione dell'header.

## Message Schema

Lo schema del messaggio è una sequenza di byte che serve a identificare il tipo di struttura dati ed il significato dei valori del payload. La sequenza di byte può essere intepretata come valore numerico o come stringa.  
La lunghezza dello schema è variabile tra 0 e 8 byte e dipende da quanto indicato nell'apposita sezione dell'header.

## Payload

Il payload è composto da una sequenza di parametri suddivisi in coppie chiave e valore.

La chiave ha dimensione 2 byte, mentre il valore ha dimensione variabile.

I primi 4 bit della chiave indicano il tipo del valore:

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

I successivi 12 bit della chiave indicano l'ID del parametro, che può essere un numero intero da 0 a 4095.

Il valore del parametro è specificato in base al tipo indicato nella chiave.

### bool

Il tipo `bool` ammette due valori:

- `0x00`: false
- `0x01`: true

### int8

Il tipo `int8` richiede 1 byte e ammette un valore con segno da `-127` a `127`.

### int16

Il tipo `int16` richiede 2 byte e ammette un valore con segno da `-32_767` a `32_767`.

### int32

Il tipo `int32` richiede 4 byte e ammette un valore con segno da `-2_147_483_647` a `2_147_483_647`.

### int64

Il tipo `int64` richiede 8 byte e ammette un valore con segno da `-9_223_372_036_854_775_807` a `9_223_372_036_854_775_807`.

### float

Il tipo `float` richiede 4 byte e ammette un valore con segno da //TODO.

### double

Il tipo `double` richiede 8 byte e ammette un valore con segno da //TODO.

### short binary

Il tipo `short binary` è formato da un numero variabile di byte.  
Il primo byte indica la lunghezza in byte del valore (da 0 a 255), seguito dalla sequenza di byte.

### medium binary

Il tipo `medium binary` è formato da un numero variabile di byte.
I primi 2 byte indicano la lunghezza in byte del valore (da 0 a 65_535), seguito dalla sequenza di byte.

### long binary

Il tipo `long binary` è formato da un numero variabile di byte.  
I primi 4 byte indicano la lunghezza in byte del valore (da 0 a 4_294_967_295), seguito dalla sequenza di byte.

### short text

Il tipo `short text` è formato da un numero variabile di byte.  
Il primo byte indica la lunghezza in byte del valore (da 0 a 255), seguito dalla sequenza di byte.  
Strutturalmente uguale a `short binary`, il valore va interpretato come stringa UTF-8.

### medium text

Il tipo `medium text` è formato da un numero variabile di byte.  
I primi 2 byte indicano la lunghezza in byte del valore (da 0 a 65_535), seguito dalla sequenza di byte.  
Strutturalmente uguale a `medium binary`, il valore va interpretato come stringa UTF-8.

### long text

Il tipo `long text` è formato da un numero variabile di byte.  
I primi 4 byte indicano la lunghezza in byte del valore (da 0 a 4_294_967_295), seguito dalla sequenza di byte.  
Strutturalmente uguale a `long binary`, il valore va interpretato come stringa UTF-8.

### array

Il tipo `array` è formato da un numero variabile di byte.
I primi 2 byte indicano il tipo degli elementi e la lunghezza dell'array.
Il formato dei primi 2 byte è simile a quello della chiave dei parametri del payload:

- i primi 4 bit indicano il tipo degli elementi dell'array
- i successivi 12 bit indicano il numero di elementi nell'array (da 0 a 4095)

### map

Il tipo `map` è formato da un numero variabile di byte.  
I primi 2 byte indicano la lunghezza della mappa (da 0 a 65_535).  
La successiva sequenza di byte è una sequenza di parametri, per un numero equivalente alla lunghezza indicata nei primi due byte, che rappresenta le coppie chiave-valore della mappa.  
La sequenza di parametri della mappa è strutturalmente equivalente a quella del payload principale; puà pertanto contenere a sua volta sotto-strutture di tipo `array` e `map`.

## CRC

Il CRC è una sequenza di 4 byte (CRC-32) posta al termine del messaggio.
La presenza del CRC è opzionale ed è indicata nell'apposito bit dell'header.

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
