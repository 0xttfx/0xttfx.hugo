---
title: "RTFM & Aleatoriedades - OCSPStapling"
date: 2024-01-21T20:30:59-03:00
layout: "simple"
# weight: 1
# aliases: ["/first"]
tags: ["Linux","TLS"]
author: "Faioli a.k.a 0xttfx"
toc: true
hideSummary: false
comments: true
#showComments: true
showDate: true
showTitle: true
showShare: true
#disableShare: false
norss: false
nosearch: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
#UseHugoToc: true
---

OSCP Stapling é o servidor Web (e não o navegador) obtendo resposta OCSP com status do certificado armazenando-o em cache, e esses dados são incluídos nas respostas HTTPS junto com o certificado...

O   **OCSP** - Online Certificate Status Protocol, descrita na [RFC 2560](https://datatracker.ietf.org/doc/rfc2560), é um protocolo útil para determinar o status de um certificado digital X.509 [RFC5280](https://datatracker.ietf.org/doc/html/rfc5280) sem exigir **CRL** - Certificate Revocation Lists  descrita na [RFC 2560](http://datatracker.ietf.org/doc/rfc2560/) ou como complemento, pois fornece informações *oportunas* sobre o status de revogação de um certificado ([RFC2459 Seção 3.3](https://datatracker.ietf.org/doc/rfc2459/)) que são utilizados por exemplo por transferências de fundos de alto valor ou grandes negociações de ações.

- O cliente OCSP emite uma solicitação de status para um *OCSP responder* e suspende a aceitação do certificado em questão até que o *responder* forneça uma resposta.

- Uma OCSP request contém os seguintes dados
	- versão do protocolo
	- requisição de serviço
	- identificador do certificado de destino
	- extensões opcionais que PODEM ser processadas pelo OCSP Responder


```bash
$ openssl s_client -status -connect 0xttfx.github.io:443
CONNECTED(00000003)
depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root CA
verify return:1
depth=1 C = US, O = DigiCert Inc, CN = DigiCert TLS RSA SHA256 2020 CA1
verify return:1
depth=0 C = US, ST = California, L = San Francisco, O = "GitHub, Inc.", CN = *.github.io
verify return:1
OCSP response: 
======================================
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: B76BA2EAA8AA848C79EAB4DA0F98B2C59576B9F4
    Produced At: Jan 11 12:13:07 2024 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: E4E395A229D3D4C1C31FF0980C0B4EC0098AABD8
      Issuer Key Hash: B76BA2EAA8AA848C79EAB4DA0F98B2C59576B9F4
      Serial Number: 044D72D77CDDA702DD5A67F2A23BBDD9
    Cert Status: good
    This Update: Jan 11 11:57:01 2024 GMT
    Next Update: Jan 18 10:57:01 2024 GMT

    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        15:e9:8c:5a:f3:19:e5:22:9a:a1:63:7c:c6:f1:fd:18:34:f3:
        ea:20:2b:8c:93:63:50:29:17:99:a4:45:72:77:6b:56:8f:68:
        f4:78:2b:b6:b2:9d:de:09:ac:1f:df:e4:fb:7d:03:9e:7b:aa:
        77:ca:58:bf:4b:0a:2d:08:08:ff:ed:7a:49:03:3c:87:08:08:
        df:1b:be:bc:62:5a:42:fc:32:be:bb:46:7e:1b:ac:6d:a1:e8:
        f8:38:da:7d:bc:dd:e4:bb:1b:09:ce:e5:1e:4a:97:92:01:f4:
        4b:ac:2b:d0:2c:5b:14:d2:29:26:2b:a7:9d:a7:10:93:22:cc:
        f4:b8:11:66:a4:34:5e:35:c3:2e:0d:e7:38:0c:ae:c1:15:2f:
        32:f3:73:59:fb:9c:9c:6d:24:63:e5:7d:54:24:60:ed:a6:bc:
        6a:a1:95:49:f1:fc:29:bf:1a:92:9d:a4:a0:0d:e3:df:fd:79:
        76:21:76:c1:cf:cd:8e:fa:3d:89:d9:1f:be:39:1a:44:5a:1b:
        89:c1:ff:27:ec:37:f2:8b:ae:b2:e7:08:dd:ee:ca:c0:28:ca:
        9a:5b:a9:fe:df:fe:9c:94:dd:fb:1f:44:b2:6b:b3:6e:ac:c3:
        24:ef:1a:63:e7:3b:dd:1e:6f:29:21:a3:c0:a7:9e:c3:6a:6a:
        fd:a8:e1:21
======================================
---
Certificate chain
 0 s:C = US, ST = California, L = San Francisco, O = "GitHub, Inc.", CN = *.github.io
   i:C = US, O = DigiCert Inc, CN = DigiCert TLS RSA SHA256 2020 CA1
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Feb 21 00:00:00 2023 GMT; NotAfter: Mar 20 23:59:59 2024 GMT
 1 s:C = US, O = DigiCert Inc, CN = DigiCert TLS RSA SHA256 2020 CA1
   i:C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root CA
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Apr 14 00:00:00 2021 GMT; NotAfter: Apr 13 23:59:59 2031 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIHEjCCBfqgAwIBAgIQBE1y13zdpwLdWmfyoju92TANBgkqhkiG9w0BAQsFADBP
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMSkwJwYDVQQDEyBE
aWdpQ2VydCBUTFMgUlNBIFNIQTI1NiAyMDIwIENBMTAeFw0yMzAyMjEwMDAwMDBa
Fw0yNDAzMjAyMzU5NTlaMGcxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9y
bmlhMRYwFAYDVQQHEw1TYW4gRnJhbmNpc2NvMRUwEwYDVQQKEwxHaXRIdWIsIElu
Yy4xFDASBgNVBAMMCyouZ2l0aHViLmlvMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAuLBgDhov8bGGS2TsEZ+meb7oh/GIxbRJmxC7yq/qr75UDHhDf8p7
TkVbCyQp8bsj/Bmkx2xwSXZT0wkjZbJIe7Ycqgca4nka+Xpe5xb4pkrVOaPiDfdX
7+34CHZbUtqL0OYebi/5D5lLalLKNOGkySAz05foenfFAxAmQYJhR6KvxFY/dqI4
y7JwrnJ6Q8F+J6Ne1uP256UwcL0qlid6e/tA0ld3ryMSJ0I6xgtqjL26Le4/nxXu
YlekppVQr0OwrHa44Q7Z/1bsdFCGtR+WLNGVBeW3BWeTTp7yWjgfp49DWt48V9pI
elDGiDgVyJcsLOz4OQk2vRmNA1ZBZgck4wIDAQABo4ID0DCCA8wwHwYDVR0jBBgw
FoAUt2ui6qiqhIx56rTaD5iyxZV2ufQwHQYDVR0OBBYEFI0CHHVazcamQXhpKMP3
qqeYO9W7MHsGA1UdEQR0MHKCCyouZ2l0aHViLmlvgglnaXRodWIuaW+CDCouZ2l0
aHViLmNvbYIKZ2l0aHViLmNvbYIOd3d3LmdpdGh1Yi5jb22CFyouZ2l0aHVidXNl
cmNvbnRlbnQuY29tghVnaXRodWJ1c2VyY29udGVudC5jb20wDgYDVR0PAQH/BAQD
AgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjCBjwYDVR0fBIGHMIGE
MECgPqA8hjpodHRwOi8vY3JsMy5kaWdpY2VydC5jb20vRGlnaUNlcnRUTFNSU0FT
SEEyNTYyMDIwQ0ExLTQuY3JsMECgPqA8hjpodHRwOi8vY3JsNC5kaWdpY2VydC5j
b20vRGlnaUNlcnRUTFNSU0FTSEEyNTYyMDIwQ0ExLTQuY3JsMD4GA1UdIAQ3MDUw
MwYGZ4EMAQICMCkwJwYIKwYBBQUHAgEWG2h0dHA6Ly93d3cuZGlnaWNlcnQuY29t
L0NQUzB/BggrBgEFBQcBAQRzMHEwJAYIKwYBBQUHMAGGGGh0dHA6Ly9vY3NwLmRp
Z2ljZXJ0LmNvbTBJBggrBgEFBQcwAoY9aHR0cDovL2NhY2VydHMuZGlnaWNlcnQu
Y29tL0RpZ2lDZXJ0VExTUlNBU0hBMjU2MjAyMENBMS0xLmNydDAJBgNVHRMEAjAA
MIIBfgYKKwYBBAHWeQIEAgSCAW4EggFqAWgAdwB2/4g/Crb7lVHCYcz1h7o0tKTN
uyncaEIKn+ZnTFo6dAAAAYZ0gHV7AAAEAwBIMEYCIQCqfmfSO8MxeeVZ/fJzqqBB
p+VqeRDUOUBVGyTTOn43ewIhAJT0S27mmGUlpqNiDADP+Jo8C6kYHF+7U6TY74bH
XHAaAHYAc9meiRtMlnigIH1HneayxhzQUV5xGSqMa4AQesF3crUAAAGGdIB1agAA
BAMARzBFAiEAguB+XQVANBj2MPcJzbz+LBPrkDDOEO3op52jdHUSW3ICIF0fnYdW
qvdtmgQNSns13pAppdQWp4/f/jerNYskI7krAHUASLDja9qmRzQP5WoC+p0w6xxS
ActW3SyB2bu/qznYhHMAAAGGdIB1SgAABAMARjBEAiAT/wA2qGGHSKZqBAm84z6q
E+dGPQZ1aCMY52pFSfcw8QIgP/SciuZG02X2mBO/miDT2hCp4y5d2sc7FE5PThyC
pbMwDQYJKoZIhvcNAQELBQADggEBADekGxEin/yfyWcHj6qGE5/gCB1uDI1l+wN5
UMZ2ujCQoKQceRMHuVoYjZdMBXGK0CIXxhmiIosD9iyEcWxV3+KZQ2Xl17e3N0zG
yOXx2Kd7B13ruBxQpKOO8Ez4uGpyWb5DDoretV6Pnj9aQ2SCzODedvS+phIKBmi7
d+FM70tNZ6/2csdrG5xIU6d/7XYYXPD2xkwkU1dX4UKmPa7h9ZPyavopcgE+twbx
LxoOkcXsNb/12jOV3iQSDfXDI41AgtFc694KCOjlg+UKizpemE53T5/cq37OqChP
qnlPyb6PYIhua/kgbH84ltba1xEDQ9i4UYfOMiJNZEzEdSfQ498=
-----END CERTIFICATE-----
subject=C = US, ST = California, L = San Francisco, O = "GitHub, Inc.", CN = *.github.io
issuer=C = US, O = DigiCert Inc, CN = DigiCert TLS RSA SHA256 2020 CA1
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA-PSS
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 4060 bytes and written 393 bytes
Verification: OK
---
New, TLSv1.3, Cipher is TLS_AES_128_GCM_SHA256
Server public key is 2048 bit
This TLS version forbids renegotiation.
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_128_GCM_SHA256
    Session-ID: 899A5EB90A464A1A310C86986066E2FCFB68AA146C1846809043BC0C7A739186
    Session-ID-ctx: 
    Resumption PSK: B24E53F7E7FF18D18BE22EE2ACE052DD858A2226E3F6528E0166A5C90FCA50F5
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 3600 (seconds)
    TLS session ticket:
    0000 - 1b bb 4f 78 61 17 18 f9-dd c9 5c 55 da 7e 75 f8   ..Oxa.....\U.~u.
    0010 - f7 88 9b a1 26 e4 5f 03-8f b7 3d b2 4f be df ac   ....&._...=.O...
    0020 - 24 40 e5 4f 46 0d 68 a3-e3 93 76 d0 5f bf 6d 11   $@.OF.h...v._.m.
    0030 - 8a 31 67 e8 75 5d 1f c2-5f 68 6f e9 27 4c e1 46   .1g.u].._ho.'L.F
    0040 - df 8c b2 92 e5 6f bf 0b-48 09 1b d5 b9 19 b7 9e   .....o..H.......
    0050 - 55 9c 04 b1 28 ae 4e 90-4b 6b c7 a8 0c b1 58 d2   U...(.N.Kk....X.
    0060 - 46 6a 5c 9c ca 4d ac 52-f5 74 3a 48 78 f7 22 02   Fj\..M.R.t:Hx.".
    0070 - b7 ab 64 78 c0 07 3a 62-d3 47 f2 51 48 2a b4 2e   ..dx..:b.G.QH*..
    0080 - 96 a7 df a4 cb f1 9b 42-17 a8 0f 03 ba a7 50 63   .......B......Pc
    0090 - 39 af 34 cd fe 3d b6 52-a1 f2 e9 53 b8 1e ee e1   9.4..=.R...S....

    Start Time: 1705071930
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
closed
```


A verificação de status de revogação está disponível para o certificado de entidade final.
- o protocolo v1 sobre HTTP e o tipo de resposta básica são suportados.

O status de revogação é verificado via OCSP quando pelo menos uma destas condições for verdadeira:

- endereço de URL de um OCSP responder é configurado.
- A verificação da Autoridade de Acesso à Informação (AIA) está ativada e o certificado a ser validado tem uma extensão AIA. 
	- A extensão AIA deve conter um método de acesso PKIK_AD_AD_OCSP com uma URI que indica o local HTTP do Responder do OCSP.

> [!NOTA]
> Apenas o primeiro *OCSP Responder* que é identificado na extensão AIA é consultado para o status de revogação.


Quando ativado a  verificação de URL e AIA, o URL responder é consultado primeiro.
- A ordem pode ser mudada configurando o atributo da API Global Security Kit (GSKit) , GSK_OCSP_CHECK_AIA_FIRST. 
- Consulta para um segundo responder ocorre apenas se o primeiro responder resultar em um status de revogação indeterminado.

As sessões cliente com processamento de OCSP habilitado podem solicitar a sessão do servidor para enviar uma resposta do OCSP como parte da negociação de sessão para protocolos TLS TLSv1.3 e TLSv1.2. 
- A sessão cliente processa a resposta do OCSP stapling no servidor, eliminando a necessidade do cliente consultar um do *OCSP responder* para o status de revogação de certificado. 
- Uma sessão do servidor também deve ativar o processamento de solicitação de status do certificado, para suportar o stapling do OCSP quando solicitado por um software cliente.

---
<script src="https://giscus.app/client.js"
        data-repo="0xttfx/0xttfx.github.io"
        data-repo-id="R_kgDOK3wAHw"
        data-category="BlogPostComments"
        data-category-id="DIC_kwDOK3wAH84Cnmtb"
        data-mapping="pathname"
        data-strict="1"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="top"
        data-theme="preferred_color_scheme"
        data-lang="en"
        data-loading="lazy"
        crossorigin="anonymous"
        async>
</script>

