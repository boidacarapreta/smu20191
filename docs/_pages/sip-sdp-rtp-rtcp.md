---
layout: default
title: SIP + SDP + RTP + RTCP
permalink: /sip-sdp-rtp-rtcp
---

# Arquivo de captura
Foi utilizado o aplicativo `tcpdump` para captura do arquivo ([sip_rtp_rtcp.pcapng](/sip_rtp_rtcp.pcapng)) - em formato [PCAPng](https://wiki.wireshark.org/Development/PcapNg).

# SIP

## Registro
O registro SIP está ausente neste primeiro arquivo. Será gerada uma segunda captura.

## Estabelecimento de sessão de mídia
O estabelecimento de sessão de mídia foi feito entre os UAs `iphone` e `linux`.

1. No cabeçalho (_Message Header_), o convite é feito de `linux` (campo `From`) para `iphone` (campo `To`) usando o método SIP INVITE. No corpo da mensagem (_Message Body_), o campo SDP `m` (_Media Description, name and address_) informa a lista de codecs suportados: apenas G.711 lei µ.
```
...
Session Initiation Protocol (INVITE)
    Request-Line: INVITE sip:iphone@35.198.6.100;transport=UDP SIP/2.0
    Message Header
        Via: SIP/2.0/UDP 191.36.14.86:56868;branch=z9hG4bK-524287-1---3496e5028fede1fc;rport
        Max-Forwards: 70
        Contact: <sip:linux@191.36.14.86:56868;transport=UDP>
        To: <sip:iphone@35.198.6.100;transport=UDP>
        From: <sip:linux@35.198.6.100;transport=UDP>;tag=519d5449
        Call-ID: blpzFplpENeqGT_8w9oYFQ..
        CSeq: 1 INVITE
        Allow: INVITE, ACK, CANCEL, BYE, NOTIFY, REFER, MESSAGE, OPTIONS, INFO, SUBSCRIBE
        Content-Type: application/sdp
        User-Agent: Z 5.2.19 rv2.8.99
        Allow-Events: presence, kpml, talk
        Content-Length: 161
    Message Body
        Session Description Protocol
            Session Description Protocol Version (v): 0
            Owner/Creator, Session Id (o): Z 0 0 IN IP4 191.36.14.86
            Session Name (s): Z
            Connection Information (c): IN IP4 191.36.14.86
            Time Description, active time (t): 0 0
            Media Description, name and address (m): audio 8000 RTP/AVP 0 101
            Media Attribute (a): rtpmap:101 telephone-event/8000
            Media Attribute (a): fmtp:101 0-16
            Media Attribute (a): sendrecv
```
2. O servidor `OpenSIPS` (campo `Server`) responde (temporariamente) para `linux` com `100 Giving a Try`, na busca do UA. Destaque para os campos `Call-ID` e `CSeq` com mesmo valor do convite, além da `tag` no campo `From`, identificando assim a resposta correspondente.
```
...
Session Initiation Protocol (100)
    Status-Line: SIP/2.0 100 Giving a try
    Message Header
        Via: SIP/2.0/UDP 191.36.14.86:56868;received=191.36.14.86;branch=z9hG4bK-524287-1---3496e5028fede1fc;rport=56868
        To: <sip:iphone@35.198.6.100;transport=UDP>
        From: <sip:linux@35.198.6.100;transport=UDP>;tag=519d5449
        Call-ID: blpzFplpENeqGT_8w9oYFQ..
        CSeq: 1 INVITE
        Server: OpenSIPS (2.4.5 (x86_64/linux))
        Content-Length: 0
```
