# Transport Layer Security

- [TCP 3-way handshake (4-times close)](#tcp-3-way-handshake--4-times-close)
- [TLS 1.2 Handshake](#tls-1-2-handshake)
- [TLS 1.3 Handshake](#tls-1-3-handshake)

## TCP 3-way handshake (4-times close)

```
          Client                                     Server
            |                                          |
     CLOSED |                                          | CLOSED
            |           SYN (seq=x, ack=0)             | LISTEN
            | ---------------------------------------> |
   SYN-SENT |        SYN | ACK (seq=y, ack=x+1)        |
            | <--------------------------------------- |
            |           ACK (seq=x+1, ack=y+1)         | SYN-RCVD
            | ---------------------------------------> |
ESTABLISHED |                                          | ESTABLISHED
            |                                          |
            |                                          |
            |                                          |
            |        FIN | ACK (seq=n, ack=m)          |
            | ---------------------------------------> |
 FIN-WAIT-1 |           ACK (seq=m, ack=n+1)           |
            | <--------------------------------------- |
 FIN-WAIT-2 |        FIN | ACK (seq=m, ack=n+1)        | CLOSE-WAIT
            | <--------------------------------------- |
            |           ACK (seq=n+1, ack=m+1)         | LAST-ACK
            | ---------------------------------------> |
  TIME-WAIT |                                          | CLOSED
     CLOSED |                                          |
            |                                          |
            V                                          V
```

## TLS 1.2 Handshake

```
   Client                           Server
     |                                |
     |          Client Hello          |
(1)  | -----------------------------> |
     |                                |
     |                                |
     |          Server Hello          |
     | <----------------------------- |  (2)
     |          Certificate           |
     | <----------------------------- |
     |       Server Key Exchange      |
     | <----------------------------- |
     |        Server Hello Done       |
     | <----------------------------- |
     |                                |
     |                                |
     |       Client Key Exchange      |
(3)  | -----------------------------> |
     |       Change Cipher Spec       |
     | -----------------------------> |
     |            Finished            |
     | -----------------------------> |
     |                                |
     |                                |
     |       Change Cipher Spec       |
     | <----------------------------- |  (4)
     |            Finished            |
     | <----------------------------- |
     |                                |
     |                                |
     |        Application Data        |
(5)  | -----------------------------> |
     |        Application Data        |
     | <----------------------------- |  (6)
     |                                |
     V                                V
```

## TLS 1.3 Handshake

```
   Client                           Server
     |                                |
     |          Client Hello          |
(1)  | -----------------------------> |
     |                                |
     |                                |
     |          Server Hello          |
     | <----------------------------- |  (2)
     |      Encrypted Extensions      |
     | <----------------------------- |
     |          Certificate           |
     | <----------------------------- |
     |       Certificate Verify       |
     | <----------------------------- |
     |            Finished            |
     | <----------------------------- |
     |                                |
     |                                |
     |          Certificate           |
(3)  | -----------------------------> |
     |       Certificate Verify       |
     | -----------------------------> |
     |            Finished            |
     | -----------------------------> |
     |                                |
     |                                |
     |        Application Data        |
     | -----------------------------> |
     |        Application Data        |
     | <----------------------------- |  (4)
     |                                |
     V                                V
```
