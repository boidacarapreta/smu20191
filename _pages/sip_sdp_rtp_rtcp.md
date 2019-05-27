---
layout: default
title: SIP + SDP + RTP + RTCP
permalink: /sip_sdp_rtp_rtcp.html
---

# Cenário

Há um servidor SIP com dupla função: Registrar e Proxy, cujo endereço IPv4 é `35.247.226.5`. De UAs (_User Agents_), há dois:

1. João: `<joao@192.168.0.166>`
2. Maria: `<maria@192.168.0.110>`

Ambos os UAs se registraram no SIP Registrar. Como o tráfego entre UAs e servidor possui NAT (pelo menos 1 conhecido), todo o tráfego SIP passará pelo mesmo servidor, que acumula a função de SIP Proxy. Assim, as URIs ficam assim:

1. João: `sip:joao@35.247.226.5`
2. Maria: `sip:maria@35.247.226.5`

# Arquivo de captura

Foi utilizado o aplicativo [tcpdump](https://www.tcpdump.org) para captura do tráfego - em formato [PCAPng](https://wiki.wireshark.org/Development/PcapNg):

```bash
tcpdump -n -i any -w sip_sdp_rtp_rtcp.pcapng udp
```

Esse primeiro filtro foi aplicado apenas a UDP. Na sequência, foi utilizado o aplicativo [Wireshark](https://www.wireshark.org) para selecionar apenas os protocolos SIP, SDP, RTP e RTCP da Camada de Aplicação, com o seguinte filtro:

```
sip or sdp or rtp or rtcp
```

Ao final, foi obtido o arquivo [sip_sdp_rtp_rtcp.pcapng]({{ "/sip_sdp_rtp_rtcp.pcapng" | relative_url }}). A seguir, será usado formato gerado pelo aplicativo Wireshark para apresentar as mensagens trafegadas entre as aplicações.

# [SIP](https://tools.ietf.org/html/rfc3261)

Para facilitar a compreensão de cada protocolo, os cabeçalhos SIP e SDP foram separados nesta documentação, porém mencionadas as relações entre os protocolos quando necessário.

## Registro

Para fins de registro, foi documentado apenas o registro do agente `joao`.

- A requisição (`REGISTER`):

```sip
Session Initiation Protocol (REGISTER)
    Request-Line: REGISTER sip:35.247.226.5 SIP/2.0
    Message Header
        Via: SIP/2.0/UDP 192.168.0.166:52142;rport;branch=z9hG4bKPjMYSRMWW2hHKTciwAW7UtzshUYELOmY9B
        Max-Forwards: 70
        From: "Jo\303\243o" <sip:joao@35.247.226.5>;tag=qZhcm1DZLe.2mge5z9911MOjGmjBCNzF
        To: "Jo\303\243o" <sip:joao@35.247.226.5>
        Call-ID: KQYP1FbpE6pso.1P8Wo.zAQts7JkgrrD
        CSeq: 54353 REGISTER
        User-Agent: Telephone 1.4
        Contact: "Jo\303\243o" <sip:joao@192.168.0.166:52142;ob>
        Expires: 300
        Allow: PRACK, INVITE, ACK, BYE, CANCEL, UPDATE, INFO, SUBSCRIBE, NOTIFY, REFER, MESSAGE, OPTIONS
        Content-Length:  0
```

- A seguir, a resposta com confirmação do servidor SIP Registrar (`200 OK`):

```sip
Session Initiation Protocol (200)
    Status-Line: SIP/2.0 200 OK
    Message Header
        Via: SIP/2.0/UDP 192.168.0.166:52142;received=177.16.145.108;rport=52142;branch=z9hG4bKPjMYSRMWW2hHKTciwAW7UtzshUYELOmY9B
        From: "Jo\303\243o" <sip:joao@35.247.226.5>;tag=qZhcm1DZLe.2mge5z9911MOjGmjBCNzF
        To: "Jo\303\243o" <sip:joao@35.247.226.5>;tag=cb89901742d7fa34dfcce4fffa472cde.d080
        Call-ID: KQYP1FbpE6pso.1P8Wo.zAQts7JkgrrD
        CSeq: 54353 REGISTER
        Contact: <sip:joao@192.168.0.166:52142;ob>;expires=300
        Server: OpenSIPS (2.4.5 (x86_64/linux))
        Content-Length: 0
```

Apesar do registro SIP não possuir tecnicamente um diálogo para tal, conforme a [RFC 3261](https://tools.ietf.org/html/rfc3261#), percebe-se a relação entre as duas mensagens na discriminação da transação: os campos `Call-ID` e `CSeq` e as _tags_ dos campos `From` e `To`.

Com o registro realizado, é possível realizar o estabelecimento de sessão de mídia entre os UAs.

## Estabelecimento de sessão de mídia

O estabelecimento de sessão de mídia foi feito entre os UAs `joao` e `maria`:

- No cabeçalho (_Message Header_), o convite é feito de `maria` (campo `From`) para `joao` (campo `To`) usando o método SIP INVITE:

```
Session Initiation Protocol (INVITE)
    Request-Line: INVITE sip:joao@177.16.145.108:52142;ob SIP/2.0
    Message Header
        Record-Route: <sip:35.247.226.5;lr>
        Via: SIP/2.0/UDP 35.247.226.5:5060;branch=z9hG4bKb7c5.e34f88c5.0
        Via: SIP/2.0/UDP 192.168.0.110:52466;received=177.16.145.108;branch=z9hG4bK.XpX5icLLQ;rport=52466
        From: "Maria" <sip:maria@35.247.226.5>;tag=CJS0LqDpj
        To: "joao" <sip:joao@35.247.226.5>
        CSeq: 20 INVITE
        Call-ID: gTbOAFKVjd
        Max-Forwards: 69
        Supported: replaces, outbound, gruu
        Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, NOTIFY, MESSAGE, SUBSCRIBE, INFO, UPDATE
        Content-Type: application/sdp
        Content-Length: 271
        Contact: <sip:maria@177.16.145.108:52466;transport=udp>;expires=3600;+sip.instance="<urn:uuid:6c9aa43a-6f33-4095-bd21-4d3c25434567>";+org.linphone.specs=groupchat
        User-Agent: Linphone_iPhone.SE_iOS12.2/4.0.2-2-gf18fb2a09 (belle-sip/1.6.3)
```

- O servidor `OpenSIPS` (campo `Server`) responde (temporariamente) para `maria` com `100 Giving a Try`, na busca do UA. Destaque para os campos `Call-ID` e `CSeq` com mesmo valor do convite, além da `tag` no campo `From`, identificando assim a resposta correspondente:

```
Session Initiation Protocol (100)
    Status-Line: SIP/2.0 100 Trying
    Message Header
        Via: SIP/2.0/UDP 35.247.226.5:5060;received=35.247.226.5;branch=z9hG4bKb7c5.e34f88c5.0
        Via: SIP/2.0/UDP 192.168.0.110:52466;rport=52466;received=177.16.145.108;branch=z9hG4bK.XpX5icLLQ
        Record-Route: <sip:35.247.226.5;lr>
        Call-ID: gTbOAFKVjd
        From: "Maria" <sip:maria@35.247.226.5>;tag=CJS0LqDpj
        To: "joao" <sip:joao@35.247.226.5>
        CSeq: 20 INVITE
        Content-Length:  0
```

- Uma vez localizado o UA na base local de registros, a requisição `INIVITE` foi encaminhada para o mesmo, e outra resposta intermediária foi enviada para `maria`, a de `180 Ringing`:

```sip
Session Initiation Protocol (180)
    Status-Line: SIP/2.0 180 Ringing
    Message Header
        Via: SIP/2.0/UDP 35.247.226.5:5060;received=35.247.226.5;branch=z9hG4bKb7c5.e34f88c5.0
        Via: SIP/2.0/UDP 192.168.0.110:52466;rport=52466;received=177.16.145.108;branch=z9hG4bK.XpX5icLLQ
        Record-Route: <sip:35.247.226.5;lr>
        Call-ID: gTbOAFKVjd
        From: "Maria" <sip:maria@35.247.226.5>;tag=CJS0LqDpj
        To: "joao" <sip:joao@35.247.226.5>;tag=NME8jLxUrC3h6Ia4DJ-GW74aGYHSJSqr
        CSeq: 20 INVITE
        Contact: <sip:joao@177.16.145.108:52142;ob>
        Allow: PRACK, INVITE, ACK, BYE, CANCEL, UPDATE, INFO, SUBSCRIBE, NOTIFY, REFER, MESSAGE, OPTIONS
        Content-Length:  0
```

- Quando `joao` atende a solicitação, confirmando o estabelecimento da sessão, a resposta é definitiva, com `200 OK`:

```sip
Session Initiation Protocol (200)
    Status-Line: SIP/2.0 200 OK
    Message Header
        Via: SIP/2.0/UDP 35.247.226.5:5060;received=35.247.226.5;branch=z9hG4bKb7c5.e34f88c5.0
        Via: SIP/2.0/UDP 192.168.0.110:52466;rport=52466;received=177.16.145.108;branch=z9hG4bK.XpX5icLLQ
        Record-Route: <sip:35.247.226.5;lr>
        Call-ID: gTbOAFKVjd
        From: "Maria" <sip:maria@35.247.226.5>;tag=CJS0LqDpj
        To: "joao" <sip:joao@35.247.226.5>;tag=NME8jLxUrC3h6Ia4DJ-GW74aGYHSJSqr
        CSeq: 20 INVITE
        Allow: PRACK, INVITE, ACK, BYE, CANCEL, UPDATE, INFO, SUBSCRIBE, NOTIFY, REFER, MESSAGE, OPTIONS
        Contact: <sip:joao@177.16.145.108:52142;ob>
        Supported: replaces, 100rel, norefersub
        Content-Type: application/sdp
        Content-Length:   277
```

- E para completar o estabelecimento da sessão, a última mensagem do _3-way handshake_, o método `ACK`:

```sip
Session Initiation Protocol (ACK)
    Request-Line: ACK sip:joao@177.16.145.108:52142;ob SIP/2.0
    Message Header
        Via: SIP/2.0/UDP 35.247.226.5:5060;branch=z9hG4bKb7c5.e34f88c5.2
        Via: SIP/2.0/UDP 192.168.0.110:52466;received=177.16.145.108;rport=52466;branch=z9hG4bK.cGe89hYZY
        From: "Maria" <sip:maria@35.247.226.5>;tag=CJS0LqDpj
        To: "joao" <sip:joao@35.247.226.5>;tag=NME8jLxUrC3h6Ia4DJ-GW74aGYHSJSqr
        CSeq: 20 ACK
        Call-ID: gTbOAFKVjd
        Max-Forwards: 69
        User-Agent: Linphone_iPhone.SE_iOS12.2/4.0.2-2-gf18fb2a09 (belle-sip/1.6.3)
```

Percebe-se que em todas as mensagens os campos `Call-ID` e `CSeq` mantêm-se inalterados: enquanto que `Call-ID` será o mesmo para todas as mensagens dessa sessão (diálogo), `CSeq` será incrementado para cada transação - no caso, todas as mensagens acima foram uma única transação. Cabe destacar que a _tag_ de `From` apareceu em todas as mensagens, enquanto que no campo _To_ foi acrescida quando localizado o UA de destino.

# [SDP](https://tools.ietf.org/html/rfc4566)

Conforme dito anteriormente, o protocolo SDP foi suprimido da seção anterior para facilitar a leitura do cabeçalho SIP, nesta seção, será mantido o SIP e adicionado o corpo da mensagem, o SDP.

## Estabelecimento de sessão de mídia

Nem todas as mensagens contêm o corpo da mensagem com SDP. Aqui serão informadas apenas as duas mensagens, a requisição (`INVITE`) e resposta com sucesso `200 OK`):

- Na requisição feita por `maria`, no corpo da mensagem (_Message Body_), o campo SDP `m` (_Media Description, name and address_) informa o tipo de mídia (áudio), a porta (`7426`/UDP) e a lista de codecs suportados (G.711 lei µ - RTP/AVP tipo 0, G.711 lei A - RTP/AVP tipo 8 e DMTF - RTP/AVP tipo 101); o campo `c` (_Connection Information_), o endereço IPv4 `192.168.0.110`:

```sdp
Session Initiation Protocol (INVITE)
    Request-Line: INVITE sip:joao@177.16.145.108:52142;ob SIP/2.0
    Message Header
        Record-Route: <sip:35.247.226.5;lr>
        Via: SIP/2.0/UDP 35.247.226.5:5060;branch=z9hG4bKb7c5.e34f88c5.0
        Via: SIP/2.0/UDP 192.168.0.110:52466;received=177.16.145.108;branch=z9hG4bK.XpX5icLLQ;rport=52466
        From: "Maria" <sip:maria@35.247.226.5>;tag=CJS0LqDpj
        To: "joao" <sip:joao@35.247.226.5>
        CSeq: 20 INVITE
        Call-ID: gTbOAFKVjd
        Max-Forwards: 69
        Supported: replaces, outbound, gruu
        Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, NOTIFY, MESSAGE, SUBSCRIBE, INFO, UPDATE
        Content-Type: application/sdp
        Content-Length: 271
        Contact: <sip:maria@177.16.145.108:52466;transport=udp>;expires=3600;+sip.instance="<urn:uuid:6c9aa43a-6f33-4095-bd21-4d3c25434567>";+org.linphone.specs=groupchat
        User-Agent: Linphone_iPhone.SE_iOS12.2/4.0.2-2-gf18fb2a09 (belle-sip/1.6.3)
    Message Body
        Session Description Protocol
            Session Description Protocol Version (v): 0
            Owner/Creator, Session Id (o): maria 126 3472 IN IP4 192.168.0.110
            Session Name (s): Talk
            Connection Information (c): IN IP4 192.168.0.110
            Time Description, active time (t): 0 0
            Session Attribute (a): rtcp-xr:rcvr-rtt=all:10000 stat-summary=loss,dup,jitt,TTL voip-metrics
            Media Description, name and address (m): audio 7246 RTP/AVP 0 8 101
            Media Attribute (a): rtpmap:101 telephone-event/8000
            Media Attribute (a): rtcp-fb:* trr-int 1000
            Media Attribute (a): rtcp-fb:* ccm tmmbr
```

- A confirmação por parte de `joao` retorna como tipo de mídia (`m`) áudio na porta `4000`/UDP e suporte a apenas G.711 lei µ - RTP/AVP tipo 0; em caminho (`c`), o endereço `192.168.0.166`:

```sdp
Session Initiation Protocol (200)
    Status-Line: SIP/2.0 200 OK
    Message Header
        Via: SIP/2.0/UDP 35.247.226.5:5060;received=35.247.226.5;branch=z9hG4bKb7c5.e34f88c5.0
        Via: SIP/2.0/UDP 192.168.0.110:52466;rport=52466;received=177.16.145.108;branch=z9hG4bK.XpX5icLLQ
        Record-Route: <sip:35.247.226.5;lr>
        Call-ID: gTbOAFKVjd
        From: "Maria" <sip:maria@35.247.226.5>;tag=CJS0LqDpj
        To: "joao" <sip:joao@35.247.226.5>;tag=NME8jLxUrC3h6Ia4DJ-GW74aGYHSJSqr
        CSeq: 20 INVITE
        Allow: PRACK, INVITE, ACK, BYE, CANCEL, UPDATE, INFO, SUBSCRIBE, NOTIFY, REFER, MESSAGE, OPTIONS
        Contact: <sip:joao@177.16.145.108:52142;ob>
        Supported: replaces, 100rel, norefersub
        Content-Type: application/sdp
        Content-Length:   277
    Message Body
        Session Description Protocol
            Session Description Protocol Version (v): 0
            Owner/Creator, Session Id (o): - 3765805867 3765805868 IN IP4 192.168.0.166
            Session Name (s): pjmedia
            Bandwidth Information (b): AS:84
            Time Description, active time (t): 0 0
            Session Attribute (a): X-nat:0
            Media Description, name and address (m): audio 4000 RTP/AVP 0 101
            Connection Information (c): IN IP4 192.168.0.166
            Bandwidth Information (b): TIAS:64000
            Media Attribute (a): rtcp:4001 IN IP4 192.168.0.166
            Media Attribute (a): sendrecv
            Media Attribute (a): rtpmap:0 PCMU/8000
            Media Attribute (a): rtpmap:101 telephone-event/8000
            Media Attribute (a): fmtp:101 0-16
```

Como houve a confirmação da requisição por parte de `joao` (`200 OK`), o áudio no sentido `maria` -> `joao` foi confirmado. Com a requisição `ACK` o UA `maria` aceita o áudio no sentido contrário `joao` -> `maria`.

## Codecs

Os codecs ofertados seguiram a [RFC 3551](https://tools.ietf.org/html/rfc3551) (seções [4.5.14](https://tools.ietf.org/html/rfc3551#section-4.5.14) e [6](https://tools.ietf.org/html/rfc3551#section-6)), e foram aceitados por ambas as partes:

- De `maria` para `joao`: G.711 leis µ e A, ou PCMU e PCMA;
- De `joao` para `maria`: apenas G.711 leis µ, ou PCMU.

## Caminhos

O cenário contempla apenas IPv4, devido a restrições da nuvem pública escolhida ([GCP](https://issuetracker.google.com/issues/35904387)). Assim a oferta de endereços se limitou a essa pilha:

- De `maria` para `joao`: `192.168.0.110`, porta `7426`/UDP;
- De `joao` para `maria`: `192.168.0.166`, porta `4000`/UDP.

# [RTP](https://tools.ietf.org/html/rfc3550)

Como ambos os UAs estavam em mesma rede local (`192.168.0.0/24`), o tráfego de mídia ocorreu diretamente entre os agentes, sendo usada a mesma porta para transmissão e recepção em cada terminal.

## Tipo de _payload_

Nas mensagens descritas a seguir, foram mantidos os cabeçalhos de Rede e de Transporte para validar os valores do SDP:

- A primeira mensagem RTP de `maria` para `joao`:

```rtp
Internet Protocol Version 4, Src: 192.168.0.110, Dst: 192.168.0.166
User Datagram Protocol, Src Port: 7246, Dst Port: 4000
Real-Time Transport Protocol
    [Stream setup by SDP (frame 8)]
    10.. .... = Version: RFC 1889 Version (2)
    ..0. .... = Padding: False
    ...0 .... = Extension: False
    .... 0000 = Contributing source identifiers count: 0
    0... .... = Marker: False
    Payload type: ITU-T G.711 PCMU (0)
    Sequence number: 0
    [Extended sequence number: 65536]
    Timestamp: 1672021947
    Synchronization Source identifier: 0x21176007 (555180039)
    Payload: ffffffffffffffffffffffffffff7eff7eff7eff7e7efe7d…
```

- Assim como de `joao` para `maria`:

```rtp
Internet Protocol Version 4, Src: 192.168.0.166, Dst: 192.168.0.110
User Datagram Protocol, Src Port: 4000, Dst Port: 7246
Real-Time Transport Protocol
    [Stream setup by SDP (frame 8)]
    10.. .... = Version: RFC 1889 Version (2)
    ..0. .... = Padding: False
    ...0 .... = Extension: False
    .... 0000 = Contributing source identifiers count: 0
    1... .... = Marker: True
    Payload type: ITU-T G.711 PCMU (0)
    Sequence number: 5234
    [Extended sequence number: 70770]
    Timestamp: 160
    Synchronization Source identifier: 0x43fe689e (1140746398)
    Payload: ffffffffffffffffffffffffffffffffffffffffffffffff…
```

## SSRC e _timestamp_

Como há apenas uma mídia, de áudio, o SSRC é único em cada sentido:

- De `maria` para `joao`: `0x21176007`;
- De `maria` para `joao`: `0x43fe689e`;

# RTCP

Em relação ao tempo de transporte da mídia, a média de jitter é de 5ms, um bom valor considerando rede local sobre IEEE 802.11ac (5GHz) com visada e curta distância (~3 metros). Em relação a perda, **não houve qualquer perda em ambos os sentidos**, valor confirmado em todos os relatórios RTCP enviados pelos UAs, totalizando:

- 1087 mensagens RTP de `joao` para `maria`;
- 1074 mensagens RTP de `maria` para `joao`.
