---
title: "RTFM & Aleatoriedades - Email Phishing"
date: 2024-12-08T23:22:12-03:00
layout: "simple"
# weight: 1
# aliases: ["/first"]
tags: ["Linux", "Cybersecurity", "Pishing", "Email", "SMTP", "IMF", "SPF", "DKIM" ]
author: "Faioli a.k.a 0xttfx"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Analisando E-mail Fishing"
#canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

Os e-mails phishing continuam fortes no radar de ameaças no cenário de cibersegurança. Atacantes explorando principalmente **anexos** ou **links** maliciosos como descrito em [MITRE ATT&CK](https://attack.mitre.org/techniques/T1566/002) nas técnicas aplicadas para forjar comunicações legítimas para roubar credenciais, instalar malware ou realizar outros tipos de fraudes; continuam sendo executados com uma alta taxa de sucesso.

Apesar do assunto já ter sido amplamente discutido na Internet! Nesse documento eu tento condensar as informações mínimas para um *Go Hose* um pouco mais descente!rs :)  

Cada e-mail é composto por dois elementos principais:

1. **Cabeçalhos**: Contêm informações sobre o envio, como remetente, destinatário e servidores envolvidos.
2. **Corpo da mensagem**: Inclui o conteúdo propriamente dito, que pode ser texto simples, HTML, links ou anexos.

A inspeção dessas partes é fundamental para identificar indícios de phishing e coletar informações úteis para investigações. Porém, na prática, é necessário compreender quais aspectos devemos aferir e como correlacioná-los com os padrões técnicos que regulamentam a troca de e-mails.



### **Padrões Fundamentais**

#### 1. Transporte de Mensagens:

- SMTP - [RFC 5321](https://datatracker.ietf.org/doc/html/rfc5321)

O **Simple Mail Transfer Protocol (SMTP)** é responsável pelo envio e roteamento de e-mails entre servidores. Ele opera como um envelope que contém:

- **MailFrom**: O endereço do remetente usado para roteamento.
- **.RcptTo**: O endereço do destinatár
- Os comandos SMTP (`HELO`, `MAIL FROM`, `RCPT TO`, `DATA`, etc.) são usados para transferir a mensagem de um servidor para outro.
- E claro a resolução DNS para registros MX (Mail Exchange).

#### 2. Estrutura da Mensagem: 

- IMF - [RFC 5322](https://datatracker.ietf.org/doc/html/rfc5322)

O **Internet Message Format (IMF)**,  define a estrutura interna do e-mail, incluindo:

- **Cabeçalhos principais**:
    - `From`: Endereço do remetente visível ao destinatário.
    - `To`: Endereço do destinatário.
    - `Subject`: Assunto do e-mail.
    - `Date`: Data e hora de envio.
- **Corpo da mensagem**: 
	- `Body`: O conteúdo, que pode ser texto simples ou HTML, conter anexos e geralmente codificados em MIME

Esses dois padrões (RFC 5321 e RFC 5322) operam em conjunto: o **SMTP** cuida do transporte da mensagem, enquanto o **IMF** define como ela é apresentada.

#### 3. Protocolos de Autenticação
Para combater fraudes e proteger a integridade das mensagens, protocolos adicionais foram criados:

**1. SPF -Sender Policy Framework**
- [RFC7208](https://datatracker.ietf.org/doc/html/rfc7208)
- Verifica se o servidor que enviou o e-mail está autorizado pelo domínio do remetente.
- Baseia-se em registros DNS TXT, que especificam quais IPs podem enviar e-mails em nome do domínio.

**2. DKIM - DomainKeys Identified Mail**:
- [RFC6376](https://datatracker.ietf.org/doc/html/rfc6376)
- Garante que o conteúdo da mensagem (cabeçalhos e corpo) não foi alterado durante o envio.
- Usa uma assinatura criptográfica (`DKIM-Signature`), validada com uma chave pública armazenada no DNS.

**3. DMARC - Domain-based Message Authentication, Reporting, and Conformance**:
- [RFC7489](https://datatracker.ietf.org/doc/html/rfc7489)
- Alinha os resultados de SPF e DKIM com o endereço visível no cabeçalho `From`.
- Define políticas para lidar com mensagens que falham nas validações (ex.: rejeitar ou colocar em quarentena).

### SMTP, IMF e Autenticação
Para visualizar melhor como essas partes interagem, considere o seguinte exemplo:

- Header de envio do e-mail
	- lembrando que em uma análise comum, você não terá acesso a esse cabeçalho! Pois é restrito ao MUA do usuário que fez o envio do e-mail! 
```txt
MIME-Version: 1.0
Date: Sun, 8 Dec 2024 20:03:53 -0300
Message-ID: <CAHExE1hY=_mC8PtrK26nC1EkU059HTBV-wEeQSF-=iq+wOOpqQ@mail.gmail.com>
Subject: E o pe?
From: "Thiago T. Faioli" <thiago.faioli@gmail.com>
To: "thiago.faioli@bsd.com.br" <thiago.faioli@bsd.com.br>
Content-Type: multipart/mixed; boundary="0000000000002758590628ca43cc"

--0000000000002758590628ca43cc
Content-Type: multipart/alternative; boundary="0000000000002758580628ca43ca"

--0000000000002758580628ca43ca
Content-Type: text/plain; charset="UTF-8"

Opa, Bao!

Ja melhorou dos bichos de pe?

E nao custa lembrar: cuidado <https://www.youtube.com/watch?v=DlP3FZbAXgI>!

Abraco do amigo
Faioli

--0000000000002758580628ca43ca
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<div dir=3D"ltr"><div><div><div>Opa, Bao!</div><div><br></div><div>Ja melho=
rou dos bichos de pe?<br></div><div><br></div><div>E nao custa lembrar: <a =
href=3D"https://www.youtube.com/watch?v=3DDlP3FZbAXgI" target=3D"_blank">cu=
idado</a>!<br></div><div><br>Abraco do am<font color=3D"#444444">igo</font>=
</div><font color=3D"#888888"><div><font color=3D"#444444">Faioli</font><sp=
an style=3D"color:rgb(0,0,0);font-size:10pt"> <br></span></div></font></div=
></div><div><div dir=3D"ltr" class=3D"gmail_signature" data-smartmail=3D"gm=
ail_signature"><div dir=3D"ltr"><div><div dir=3D"ltr"><div dir=3D"ltr"><div=
 dir=3D"ltr"><div dir=3D"ltr"><div><div style=3D"font-family:arial"><div st=
yle=3D"font-size:small"><div><br></div></div></div></div></div></div></div>=
</div></div></div></div></div></div>

--0000000000002758580628ca43ca--
--0000000000002758590628ca43cc
Content-Type: application/pdf; name="Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf"
Content-Disposition: attachment; filename="Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf"
Content-Transfer-Encoding: base64
X-Attachment-Id: f_m4g7m37i0
Content-ID: <f_m4g7m37i0>


--0000000000002758590628ca43cc--
```

- header do e-mail no MUA do usuário que recebeu o e-mail.
	- são esses os dados que teremos para investigação.
```txt
Delivered-To: thiago.faioli@bsd.com.br
Received: by 2002:a05:7110:4344:b0:254:86f7:fd0c with SMTP id g4csp220273gee;
        Sun, 8 Dec 2024 15:04:28 -0800 (PST)
X-Received: by 2002:a17:90b:28d0:b0:2ea:8aac:6aa9 with SMTP id 98e67ed59e1d1-2ef6a6bd802mr15278535a91.21.1733699060421;
        Sun, 08 Dec 2024 15:04:20 -0800 (PST)
ARC-Seal: i=1; a=rsa-sha256; t=1733699060; cv=none;
        d=google.com; s=arc-20240605;
        b=EtZYcJUab+vSEdZhsk/uCg1+PyzpSEmw2aXc8YPvdB2K2WWfzf33QfrV9HP6SHMm80
         IcCwNYfiPDVwkNc6TWi9UKXY5EBm12qxrS3rLwvE+hNWIbQRhlsDo3lE1YaX4yLXxEsv
         WQo6rxzGjTXRDTiGap6CeMm5XLbaZ6RkUVe5tkpok2hcgJ/Njn/aRcUJa+Tz/wKcAu6h
         y9rG56oMa2WZwexukHtHhDbwNn+NZvYYB02WEe8tlNieIzECAjEY+eI14Vt/NJVdf2hR
         hVeYAOqGDy8CLvGCQuiB6eLkD4trwY8nPQka8W5tKHvUGadDOOQcTHQpFbFfURKrajNr
         OJUw==
ARC-Message-Signature: i=1; a=rsa-sha256; c=relaxed/relaxed; d=google.com; s=arc-20240605;
        h=to:subject:message-id:date:from:mime-version:dkim-signature;
        bh=AIDAEBlTTt9xgiNB096cNNX1VJAj2rhMPqxYE23Kufo=;
        fh=Fi9mIQ4POoGfXYmpD5y/vuvHDDd4psKIIeu1RdDwRx4=;
        b=e6ONX6MjloNz1wQ+QbEkuF1wT36cfEIlCch19xZikBGsRpkDmFpymq1HVx/GTOZEGr
         50zYSAIhOoUHWSHytJp0Ges1Z2wtO4PQKG6ZUcGRlzKWCEsh10WG7/wcHbxcuEXZEUJ0
         Nx0WmdZjwzLZXZt7H+5ORr7EgG1ai+ZcMmHvxHULJRjkwLTCgyJSpo7yAHGJupvAygxK
         b43MeJi1tBBN2bIgpU0aZgjkpttMTKzEpT7NZx1LDn4NFiGTmfHWWpOE4Lyz6T4NXAbR
         7gTvqZs9CdO5K3Lst5t/etKBGb4ZrTQnndmOKfYl4e2uvkHrUiLDDGkISXdvYTlxYx6g
         Z1Wg==;
        dara=google.com
ARC-Authentication-Results: i=1; mx.google.com;
       dkim=pass header.i=@gmail.com header.s=20230601 header.b=kM+M4SQb;
       spf=pass (google.com: domain of thiago.faioli@gmail.com designates 209.85.220.41 as permitted sender) smtp.mailfrom=thiago.faioli@gmail.com;
       dmarc=pass (p=NONE sp=QUARANTINE dis=NONE) header.from=gmail.com;
       dara=neutral header.i=@bsd.com.br
Return-Path: <thiago.faioli@gmail.com>
Received: from mail-sor-f41.google.com (mail-sor-f41.google.com. [209.85.220.41])
        by mx.google.com with SMTPS id 98e67ed59e1d1-2ef7a4c2a11sor3537848a91.6.2024.12.08.15.04.19
        for <thiago.faioli@bsd.com.br>
        (Google Transport Security);
        Sun, 08 Dec 2024 15:04:19 -0800 (PST)
Received-SPF: pass (google.com: domain of thiago.faioli@gmail.com designates 209.85.220.41 as permitted sender) client-ip=209.85.220.41;
Authentication-Results: mx.google.com;
       dkim=pass header.i=@gmail.com header.s=20230601 header.b=kM+M4SQb;
       spf=pass (google.com: domain of thiago.faioli@gmail.com designates 209.85.220.41 as permitted sender) smtp.mailfrom=thiago.faioli@gmail.com;
       dmarc=pass (p=NONE sp=QUARANTINE dis=NONE) header.from=gmail.com;
       dara=neutral header.i=@bsd.com.br
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=gmail.com; s=20230601; t=1733699059; x=1734303859; darn=bsd.com.br;
        h=to:subject:message-id:date:from:mime-version:from:to:cc:subject
         :date:message-id:reply-to;
        bh=AIDAEBlTTt9xgiNB096cNNX1VJAj2rhMPqxYE23Kufo=;
        b=kM+M4SQb7wpOYf9dah/WaTaw2v3ONv+BL3POKNP9468+sZW1vookbtbq1HXV46RZxQ
         jUiOMnWuveBpU0RZ00GlOnxKqk9EkV/F9c++ckbnqmjiKqz0CSHQioZuSLTxE+3LYNK7
         3PQ0iQgC0KzTtCquq17EFXPJN7fqnUYTUsTc7DuukPGadowZ//Bk7hjny580yfaLbNI8
         KKKqOt1g8QKzDW/bJoxAvfqmtBRWl9WNVOoF2nzYD0fMAdvjs9Pak8VcScsa6hS7NZq3
         mVfFdSzEezQpoabXsPuB5kkSNcDYnIp24n4+H999Rk0nXtscfEpkRRlipdpVy9w0ePfd
         RHRQ==
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=1e100.net; s=20230601; t=1733699059; x=1734303859;
        h=to:subject:message-id:date:from:mime-version:x-gm-message-state
         :from:to:cc:subject:date:message-id:reply-to;
        bh=AIDAEBlTTt9xgiNB096cNNX1VJAj2rhMPqxYE23Kufo=;
        b=RJg97hgLLFL1LDrNEcv9Zb2xqwaId3PMtx4MyHYgqwBkzgrueOiFHiiMkT2vQXfYhE
         l/UYoADqOdmt6mn4Urc6YUt/NlkQpecb6HL1U9lD22wNu7zQszw5wpGxpaV07Gssx64q
         t7XSLAUxPCrLHrDr8ZcJKgNgwoOqfpMyfTd8ocVODrGKEPv5Xj8qr9x/rAAVgymJV+qc
         uDLweSTqO923yxTYARDMJ4Xxxk2F5i3Nc+52yt0Qrw06G1f0k+2ysmw8KxgNmihr1JxD
         IJO8YTIZhE7gny3eLAtxZoYCQQGfEXMd9vRxLsJfC7k2G57A8cGQ4rc85bJy6Ehvv5nd
         CbdA==
X-Gm-Message-State: AOJu0YzQGaC/YFdJeCJs2/6ut6XitFWInxTlDguCq+7o5DNwqhX0Qied aVSZqtyHDtq8POauNLzOW9QtfNxbJaLRYbdMjb+1Tvq1gnllR1HQVdlVUWvSh7QYOSjXe+uPaXH QLTkcrELGlUKoTVj/Ok6Xc2Byt+Pfi4q2Y9g=
X-Gm-Gg: ASbGncv9NT1Cf3YRMOSTUsJN0TPt+y+/Nxqmc/83GGYyVxVkTa8uQJOPqfj1xSD7Xfy GEa0y3kV7ygFCX/ygf14ZLkORyPl+3w==
X-Google-Smtp-Source: AGHT+IH4nJkPc6OewVhSlQuPT28xp0aTQNlvQa/GeF+4ZfTjHe8Yr2Mru2jbJZ9oTOF4ux9WWy56SUiVp8RbK1GmyOk=
X-Received: by 2002:a17:90b:53ce:b0:2ee:df70:1ff3 with SMTP id 98e67ed59e1d1-2ef69199134mr18507867a91.0.1733699054597; Sun, 08 Dec 2024 15:04:14 -0800 (PST)
MIME-Version: 1.0
From: "Thiago T. Faioli" <thiago.faioli@gmail.com>
Date: Sun, 8 Dec 2024 20:03:53 -0300
Message-ID: <CAHExE1hY=_mC8PtrK26nC1EkU059HTBV-wEeQSF-=iq+wOOpqQ@mail.gmail.com>
Subject: E o pe?
To: "thiago.faioli@bsd.com.br" <thiago.faioli@bsd.com.br>
Content-Type: multipart/mixed; boundary="0000000000004e955c0628ca440a"

--0000000000004e955c0628ca440a
Content-Type: multipart/alternative; boundary="0000000000004e955b0628ca4408"

--0000000000004e955b0628ca4408
Content-Type: text/plain; charset="UTF-8"

Opa, Bao!

Ja melhorou dos bichos de pe?

E nao custa lembrar: cuidado <https://www.youtube.com/watch?v=DlP3FZbAXgI>!

Abraco do amigo
Faioli

--0000000000004e955b0628ca4408
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<div dir=3D"ltr"><div><div><div>Opa, Bao!</div><div><br></div><div>Ja melho=
rou dos bichos de pe?<br></div><div><br></div><div>E nao custa lembrar: <a =
href=3D"https://www.youtube.com/watch?v=3DDlP3FZbAXgI" target=3D"_blank">cu=
idado</a>!<br></div><div><br>Abraco do am<font color=3D"#444444">igo</font>=
</div><font color=3D"#888888"><div><font color=3D"#444444">Faioli</font><sp=
an style=3D"color:rgb(0,0,0);font-size:10pt"> <br></span></div></font></div=
></div><div><div dir=3D"ltr" class=3D"gmail_signature" data-smartmail=3D"gm=
ail_signature"><div dir=3D"ltr"><div><div dir=3D"ltr"><div dir=3D"ltr"><div=
 dir=3D"ltr"><div dir=3D"ltr"><div><div style=3D"font-family:arial"><div st=
yle=3D"font-size:small"><div><br></div></div></div></div></div></div></div>=
</div></div></div></div></div></div>

--0000000000004e955b0628ca4408--
--0000000000004e955c0628ca440a
Content-Type: application/pdf; name="Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf"
Content-Disposition: attachment; filename="Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf"
Content-Transfer-Encoding: base64
Content-ID: <f_m4g7m37i0>
X-Attachment-Id: f_m4g7m37i0


--0000000000004e955c0628ca440a--
```



- Nesse exemplo, eu envio um email  de `@gmail.com` para `@bsd.com.br`! E no MUA do usuário, no caso o destinatário, a plataforma GMail do Google, siga esses passos para coletar o header da mensagem recebida:

1. Abra o [Gmail](https://mail.google.com/) no computador.
	1. Abra o e-mail que você quer analisar.
	2. Ao lado de Responder
       - clique em Mais. 
       - Mostrar original.
	3. O cabeçalho completo será exibido em uma nova janela.
	4. Clique em **Copiar para a área de transferência**.
	5. E para esse tutorial, vamos salvar esse conteúdo no arquivo: **header_bichodepe.eml**


#### 1. Roteamento e Autenticação

1. **`Return-Path`:**    
    - Indica o endereço usado para retorno de mensagens (bounce).
    - Geralmente corresponde ao `MailFrom` (SMTP Envelope) no protocolo RFC 5321.
2. **`Received`:**
    - Lista os servidores que manipularam a mensagem durante o envio, incluindo informações como o hostname (`mail-sor-f41.google.com`), IP (`209.85.220.41`) e timestamp.
    - Crucial para rastrear a origem real da mensagem.
3. **`Received-SPF`:**
    - Resultado da validação SPF:
        - **Pass:** O IP está autorizado a enviar mensagens para o domínio.
        - Detalha o IP (`209.85.220.41`)e o domínio validados (`google.com`).
4. **`Authentication-Results`:**
    - Contém os resultados de autenticação para **SPF**, **DKIM** e **DMARC**:
        - **SPF:** Validação do IP (`209.85.220.41`) como remetente do (`smtp.mailfrom=thiago.faioli@gmail.com`).
        - **DKIM:** Verificação da assinatura criptográfica (`header.i=gmail.com`).
        - **DMARC:** Confirma alinhamento do domínio `From` com SPF/DKIM.

#### 2. Autenticação (DKIM-Signature)

1. **`DKIM-Signature`:**
    - Assinatura criptográfica que valida a integridade da mensagem:
        - **`d`:** Domínio do remetente (`gmail.com`).
        - **`s`:** Selector usado para localizar a chave pública no DNS (`20230601`).
        - **`h`:** Campos assinados (`from`, `to`, `subject`, etc.).
        - **`b`:** Assinatura gerada.

#### 3. Cabeçalho da Mensagem (IMF - RFC 5322)

1. **`From`:**
    - Endereço do remetente exibido ao destinatário (`thiago.faioli@gmail.com`).
2. **`To`:**
    - Endereço do destinatário (`thiago.faioli@bsd.com.br`).
3. **`Subject`:**
    - Assunto do e-mail.
4. **`Date`:**
    - Data e hora de envio.
5. **`Message-ID`:**
    - Identificador único da mensagem, usado para rastreamento e respostas.

#### 4. Campos de Formato e Conteúdo

1. **`MIME-Version`:**
    - Indica a versão do MIME usada na mensagem.
2. **`Content-Type`:**
    - Especifica o tipo de conteúdo da mensagem, aqui como multipart/alternative (pode conter texto e HTML).


### Visão
Segue um diagrama pra conseguirmos tridimensionalizar todos os aspectos que vimos até aqui

![EmailDiagram](/images/EmailFishing/email-diagram.svg)

### Ok! Tudo bonito. Mas e na prática ?
Vou tentar demonstrar uma análise prática usando ferramentas Unix&Linux ;)

### Inspecionar o Cabeçalho

1. Extraia os cabeçalhos completos do e-mail:
    - Em clientes de e-mail, use as opções para exibir o "código-fonte", "Mostrar original" ou "cabeçalhos completos" etc...
	    - salve em um arquivo normalmente em `.eml`. Aqui em nosso exemplo será: `header_bichodepe.eml`.
1. Verifique inconsistências como por exemplo em:
    - `From`: Analise se o domínio está alinhado com os campos SPF/DKIM.
    - `Received`: Confirme a origem do servidor e as etapas de entrega.
    - `Reply-To`: Pode ser usado para redirecionar respostas para um endereço diferente do `From`.

- aqui estou fazendo um grep para trazer os dados que mais nos interessa
```bash
grep -iE "from:|to:|subject:|received:|spf|dkim|dmarc|authentication-results" header_bichodepe.eml

Delivered-To: thiago.faioli@bsd.com.br
Received: by 2002:a05:7110:4344:b0:254:86f7:fd0c with SMTP id g4csp220273gee;
X-Received: by 2002:a17:90b:28d0:b0:2ea:8aac:6aa9 with SMTP id 98e67ed59e1d1-2ef6a6bd802mr15278535a91.21.1733699060421;
        h=to:subject:message-id:date:from:mime-version:dkim-signature;
ARC-Authentication-Results: i=1; mx.google.com;
       dkim=pass header.i=@gmail.com header.s=20230601 header.b=kM+M4SQb;
       spf=pass (google.com: domain of thiago.faioli@gmail.com designates 209.85.220.41 as permitted sender) smtp.mailfrom=thiago.faioli@gmail.com;
       dmarc=pass (p=NONE sp=QUARANTINE dis=NONE) header.from=gmail.com;
Received: from mail-sor-f41.google.com (mail-sor-f41.google.com. [209.85.220.41])
Received-SPF: pass (google.com: domain of thiago.faioli@gmail.com designates 209.85.220.41 as permitted sender) client-ip=209.85.220.41;
Authentication-Results: mx.google.com;
       dkim=pass header.i=@gmail.com header.s=20230601 header.b=kM+M4SQb;
       spf=pass (google.com: domain of thiago.faioli@gmail.com designates 209.85.220.41 as permitted sender) smtp.mailfrom=thiago.faioli@gmail.com;
       dmarc=pass (p=NONE sp=QUARANTINE dis=NONE) header.from=gmail.com;
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        h=to:subject:message-id:date:from:mime-version:from:to:cc:subject
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        h=to:subject:message-id:date:from:mime-version:x-gm-message-state
         :from:to:cc:subject:date:message-id:reply-to;
X-Received: by 2002:a17:90b:53ce:b0:2ee:df70:1ff3 with SMTP id 98e67ed59e1d1-2ef69199134mr18507867a91.0.1733699054597; Sun, 08 Dec 2024 15:04:14 -0800 (PST)
From: "Thiago T. Faioli" <thiago.faioli@gmail.com>
Subject: E o pe?
To: "thiago.faioli@bsd.com.br" <thiago.faioli@bsd.com.br>
```

#### 1. Validando roteamento 

- Verificando os campos `Received:`
```bash
$ grep -A1 -i ^"received:" header_bichodepe.eml
Received: by 2002:a05:7110:4344:b0:254:86f7:fd0c with SMTP id g4csp220273gee;
        Sun, 8 Dec 2024 15:04:28 -0800 (PST)
--
Received: from mail-sor-f41.google.com (mail-sor-f41.google.com. [209.85.220.41])
        by mx.google.com with SMTPS id 98e67ed59e1d1-2ef7a4c2a11sor3537848a91.6.2024.12.08.15.04.19
```

- Agora verificamos o IP do servidor remetente
	- Que deve resolver para `mail-sor-f41.google.com`
	```bash
	$ host 209.85.220.41
	41.220.85.209.in-addr.arpa domain name pointer mail-sor-f41.google.com.
	```
	- Aqui validamos que o IP resolve para o domain esperado!
		- Tudo ok!

#### 2. Validar SPF

1. Validando campo `Received-SPF`
	```bash 
	$ grep -i received-spf header_bichodepe.eml

	Received-SPF: pass (google.com: domain of thiago.faioli@gmail.com designates 209.85.220.41 as permitted sender) client-ip=209.85.220.41;
	```
	- Aqui validação do remetente gmail.com para o servidor de envio designado é `PASS`. 
		- Tudo ok!
2. Localize o `include` do registro SPF do domínio remetente (DNS TXT)
	- Nós queremos validar o domínio do remetente (`gmail.com`) e o IP (`209.85.220.41`)
		- vamos então procurar pelo IP do servidor de envio na lista do registro SPF
			- e se essa validação **falhar**: é um potencial spoofing.
	- Extraindo dados
		```bash
		$ dig gmail.com TXT +noall +answer |grep spf
		
		gmail.com.		281	IN	TXT	"v=spf1 redirect=_spf.google.com"
		```
	- Identificamos que o registro **SPF** do **gmail.com** é um **redirect** para **google.com**.
		- **redirect** é um ponteiro para outro nome de domínio que hospeda uma política SPF.
			- Consultando o registro
				```bash
				$ dig _spf.google.com TXT +noall +answer
				_spf.google.com.	111	IN	TXT	"v=spf1 include:_netblocks.google.com include:_netblocks2.google.com include:_netblocks3.google.com ~all"
				```  
		-  Identificamos que o registro tem 3 `include`! 
			- **include** é um ponteiro para um domínio com registros SPF que serão consultados sobre IPs permitidos.
			- E esperamos que o nosso IP `209.85.220.41` que resolve para `mail-sor-f41.google.com` esteja contido em um range (CIDR) ou ser ele mesmo em algum desses registros que encontramos...  
				- `_netblocks.google.com`
					```bash
					$ dig _netblocks.google.com TXT +short |tr '[:blank:]' '\n'
					"v=spf1
					ip4:35.190.247.0/24
					ip4:64.233.160.0/19
					ip4:66.102.0.0/20
					ip4:66.249.80.0/20
					ip4:72.14.192.0/18
					ip4:74.125.0.0/16
					ip4:108.177.8.0/21
					ip4:173.194.0.0/16
					ip4:209.85.128.0/17
					ip4:216.58.192.0/19
					ip4:216.239.32.0/19
					~all"
					```
				- `_netblocks2.google.com`
					```bash
					$ dig _netblocks2.google.com TXT +short |tr '[:blank:]' '\n'
					"v=spf1
					ip6:2001:4860:4000::/36
					ip6:2404:6800:4000::/36
					ip6:2607:f8b0:4000::/36
					ip6:2800:3f0:4000::/36
					ip6:2a00:1450:4000::/36
					ip6:2c0f:fb50:4000::/36
					~all"
					```
				- `_netblocks3.google.com`
					```bash
					$ dig _netblocks3.google.com TXT +short |tr '[:blank:]' '\n'
					"v=spf1
					ip4:172.217.0.0/19
					ip4:172.217.32.0/20
					ip4:172.217.128.0/19
					ip4:172.217.160.0/20
					ip4:172.217.192.0/19
					ip4:172.253.56.0/21
					ip4:172.253.112.0/20
					ip4:108.177.96.0/19
					ip4:35.191.0.0/16
					ip4:130.211.0.0/22
					~all"
					```
		- Vamos verificar à quem foi concedido o nosso IP 
		```bash
		$ whois 209.85.220.41 |grep -Ei "cidr|range|originas|netname"
		NetRange:       209.85.128.0 - 209.85.255.255
		CIDR:           209.85.128.0/17
		NetName:        GOOGLE
		OriginAS:       
		```
		- Aqui já conseguimos validar que:
			- o IP do server remetente pertence ao Google
			- o IP está contido na rede `ip4:209.85.128.0/17` do include  `_netblocks.google.com`. 
			- o IP pertence à rede `209.85.128.0/17`
				- Seguro morre de velho né! Podemos usar o `ipcalc` pra mais um check.
					```bash
					$ ipcalc 209.85.220.41/17
					Address:	209.85.220.41
					Network:	209.85.128.0/17
					Netmask:	255.255.128.0 = 17
					Broadcast:	209.85.255.255
					
					Address space:	Internet
					HostMin:	209.85.128.1
					HostMax:	209.85.255.254
					Hosts/Net:	32766
				  ```
				- Como esperado: tudo ok!
 3. Agora podemos fazer um double check ;)
	- com a ferramenta **sfpquery**.
	```bash
	$ spfquery --ip=209.85.220.41 --mfrom=thiago.faioli@gmail.com
	pass
	gmail.com ... _spf.google.com: Sender is authorized to use 'thiago.faioli@gmail.com' in 'mfrom' identity (mechanism 'include:_netblocks.google.com' matched)
	gmail.com ... _spf.google.com: Sender is authorized to use 'thiago.faioli@gmail.com' in 'mfrom' identity (mechanism 'include:_netblocks.google.com' matched)
	Received-SPF: pass (gmail.com ... _spf.google.com: Sender is authorized to use 'thiago.faioli@gmail.com' in 'mfrom' identity (mechanism 'include:_netblocks.google.com' matched)) receiver=ctsme0; identity=mailfrom; envelope-from=  "thiago.faioli@gmail.com"; client-ip=209.85.220.41
	```


#### 3. Validar DKIM

1. Verifique o cabeçalho `DKIM-Signature`.

```bash
$ grep -A10 -Ei ^dkim-signature header_bichodepe.eml 
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=gmail.com; s=20230601; t=1733580554; x=1734185354; darn=bsd.com.br;
        h=to:subject:message-id:date:from:mime-version:from:to:cc:subject
         :date:message-id:reply-to;
        bh=2nLYWHA/hakwaoiHgM1K0wkbXqpu1yJbyP+j2RaZ4XE=;
        b=Ycbqe8w6ViHgyjCXGbc33lxvv5Gyx9Y4bwO69eoqoADUdiHtzfLcwmVCsONnYrD5hT
         7h2h6KPRmYpPNIPKuWb5yOkXIYF9GIGopQobBJ8eOdBn/JhZVx1nPWZVnZkk6TQQLCQK
         kRZ06W6o2JluhFk/a/RXy/5Ahu+G+92YHxUn5zwZf4ILjBgnQSjSPaLouvbqtYm8fu1O
         6iTaHhmYs6QLcWxbiUqjjhrPNlEwQXfo+D+f496t0Xj9SjKWebAXVxmeGc0VXExjrtRE
         1QEDXvVZjPwpo62q+59ZPou8kpGoexLGO7HsQ+0h2sn2EUwMfK6bmf1aS5QFjEMRrVLQ
         5R4Q==
```
- Aqui nos interessa os seletores
	- `s=20230601` que corresponde ao remetente 
	- `d=gmail.com` que corresponde ao domínio assinado.
- Que nos permite
	- testar a chave pública
		- Vamos usar a tool **opendkim-testkey**
			```bash
			# opendkim-testkey -d 'gmail.com' -s '20230601' -vvv

			opendkim-testkey: checking key '20230601._domainkey.gmail.com'
			opendkim-testkey: key not secure
			opendkim-testkey: key OK
			```
			- `Key not secure` não é um ponto de atenção para os nossos testes! Provavelmente trata-se do DNSSEC não verificado pelos registros `DNSKEY`, `RRSIG`, `DS` e `NSEC`...
				- E o resultado final é **key ok** o que indica estar tudo ok com o nosso remetente sobre a assinatura da mensagem.
	-  Consultar a chave pública para validação do remetente
		```bash
		$ dig txt 20230601._domainkey.gmail.com  +noall +answer
		20230601._domainkey.gmail.com. 3600 IN	TXT	"v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAntvSKT1hkqhKe0xcaZ0......aKUABGu4NTbR2fDKnYwiq5jQyBkLWP+LgGOgfUF4T4HZb2" "PY2bQtEP6QeqOtcW4rrsH24L7XhD+.....7tEDNmrZ5XUwQYUUADBOu7t1niwXwIDAQAB"
		```
	- salva-la em um arquivo
		```bash
		dig txt 20230601._domainkey.gmail.com  +short > gmail-dkimkey.pub
		```

2. Por último 
- podemos fazer uma segunda validação usando a ferramenta `opendkim-testmsg` de duas maneiras:
	1. usando o operador de redirecionamento de input `<` enviando o header para o comando:
	```bash
	$ opendkim-testmsg < header_bichodepe.eml

	```
	2. copie todo o conteúdo do nosso `header_bichodepe.eml`
		1. execute o **opendkim-testmsg** sem argumentos
		2. cole o conteúdo `CRTL-V` ou `SHIFT-ins` ` 
		3. em seguida termine com `CTRL-D`
			```bash
			$ opendkim-testmsg
			
			<CTRL-D>
			```
		- Note que não obtivemos nenhuma output!
			- O que significa que a assinatura foi verificada corretamente e está: tudo ok!
		- Mas se existir uma output parecido com
			- `opendkim-testmsg: dkim_eom(): Unable to verify`
			- `opendkim-testmsg: dkim_chunk(): No signature`
				- então temos algo suspeito...

#### 4. Validar DMARC

1. Confirmando o alinhamento.

- O domínio `From` deve corresponder ao domínio autenticado por SPF/DKIM.
	```bash
	$ grep -iE ^"from:" header_bichodepe.eml
	
	From: "Thiago T. Faioli" <thiago.faioli@gmail.com>
	```
	- aqui precisamos identificar o domínio que é `gmail.com` 
		- Então usamos essa informação para verificar o registro DMARC e a política (None, Quarantine, Reject).
			```bash
			$ dig TXT _dmarc.gmail.com +noall +answer
			
			_dmarc.gmail.com.	11	IN	TXT	"v=DMARC1; p=none; sp=quarantine; rua=mailto:mailauth-reports@google.com"
			```
			- legenda

            | Tag | Purpose | Sample |
            | --- | --- | --- |
            | v | Protocol version | v=DMARC1 |
            | pct | Percentage of messages subjected to filtering | pct= |
            | ruf | Reporting URI for forensic reports | ruf= |
            | rua | Reporting URI of aggregate reports | rua=mailto:mailauth-reports@google.com |
            | p | Policy for organizational domain | p=none |
            | sp | Policy for subdomains of the OD | sp=quarantine |
            | adkim |Alignment mode for DKIM| adkim= |
            | aspf | Alignment mode for SPF| aspf= |
            | ri | Reporting interval of aggregate reports (default "86400" ; often disregarded to default value)| ri= |
            | fo | Forensic report options (default "0")| fo= |

				- The alignment modes for DKIM and SPF can be:
					- "s" for strict: means "strict". Domains from From: shall match DKIM/SPF identifier.
					- "r" for relaxed: means "relaxed". Organizational domains from From: and DKIM/SPF shall match.
				- The domain policy (p) and subdomain policy (sp) might be one of:
					- "none" (for monitor mode)get at
					- "quarantine"
					- "reject"
				- The forensic report options are:
					- "0" to generate reports if all underlying authentication mechanisms (SPF and DKIM) fail to produce a DMARC pass result
					- "1" to generate reports if any mechanisms (SPF or DKIM) fail
					- "d" to generate report if the DKIM signature failed to verify
					- "s" if SPF failed.

		
		- Para então validar o resultado do alinhamento DMARC  no cabeçalho `Authentication-Results`
			```bash
			$ grep -A4 -Ei ^'authentication-results:' header_bichodepe.eml |grep dmarc
			
			dmarc=pass (p=NONE sp=QUARANTINE dis=NONE) header.from=gmail.com;
			```
2. Agora podemos fazer um double check
	- Usando a tool **opendmarc-check** que consulta o registro DMARC traduzindo o conteúdo em formulário legível por humanos.
		```bash
		$ opendmarc-check gmail.com
		
		DMARC record for gmail.com:
		Sample percentage: 100
		DKIM alignment: relaxed
		SPF alignment: relaxed
		Domain policy: none
		Subdomain policy: quarantine
		Aggregate report URIs:
			mailto:mailauth-reports@google.com
		Failure report URIs:
		(none)
		```


#### 5. Verificar URLs e Links

1. Inspecione todos os links no corpo da mensagem:
    - **Procure por domínios falsos** como por exemplo: `amaz0n.com` em vez de `amazon.com` etc...
	- Extraindo links 
	```bash
	$ grep -oP '(http|https)://[^\s]+' header_bichodepe.eml
	
	https://www.youtube.com/watch?v=DlP3FZbAXgI>!
	https://www.youtube.com/watch?v=3DDlP3FZbAXgI
	```
	- Procurar pro redirecionamentos usando o **curl**
	```bash
	$ curl -I https://www.youtube.com/watch?v=DlP3FZbAXgI
	HTTP/2 200 
	content-type: text/html; charset=utf-8
	x-content-type-options: nosniff
	cache-control: no-cache, no-store, max-age=0, must-revalidate
	pragma: no-cache
	expires: Mon, 01 Jan 1990 00:00:00 GMT
	date: Sun, 08 Dec 2024 21:59:59 GMT
	content-length: 814649
	strict-transport-security: max-age=31536000
	x-frame-options: SAMEORIGIN
	permissions-policy: ch-ua-arch=*, ch-ua-bitness=*, ch-ua-full-version=*, ch-ua-full-version-list=*, ch-ua-model=*, ch-ua-wow64=*, ch-ua-form-factors=*, ch-ua-platform=*, ch-ua-platform-version=*
	origin-trial: AmhMBR6zCLzDDxpW+HfpP67BqwIknWnyMOXOQGfzYswFmJe+fgaI6X......dxQ4AAAB4eyJvcmlnaW4iOiJodHRwczovL3lvdXR1YmUuY29tOjQ0MyIsImZlYXR1cmUiOiJXZWJWaWV3WFJlcXVlc3RlZFdpdGhEZXByZWNhdGlvbiIsImV4cGlyeSI6MTc1ODA2NzE5OSwiaXNTdWJkb21haW4iOnRydWV9
	cross-origin-opener-policy: same-origin-allow-popups; report-to="youtube_main"
	content-security-policy: require-trusted-types-for 'script'
	report-to: {"group":"youtube_main","max_age":2592000,"endpoints":[{"url":"https://csp.withgoogle.com/csp/report-to/youtube_main"}]}
	p3p: CP="This is not a P3P policy! See http://support.google.com/accounts/answer/151657?hl=en for more info."
	server: ESF
	x-xss-protection: 0
	set-cookie: GPS=1; Domain=.youtube.com; Expires=Sun, 08-Dec-2024 22:29:59 GMT; Path=/; Secure; HttpOnly
	set-cookie: YSC=TMzezdpwE9k; Domain=.youtube.com; Path=/; Secure; HttpOnly; SameSite=none
	set-cookie: VISITOR_INFO1_LIVE=eZKVIcLVEqU; Domain=.youtube.com; Expires=Fri, 06-Jun-2025 21:59:59 GMT; Path=/; Secure; HttpOnly; SameSite=none
	set-cookie: VISITOR_PRIVACY_METADATA=CgJVUxIEGgAgWQ%3D%3D; Domain=.youtube.com; Expires=Fri, 06-Jun-2025 21:59:59 GMT; Path=/; Secure; HttpOnly; SameSite=none
	alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
	```
	- Podemos verificar o link com VirusTotal
	```bash
	URL="https://www.youtube.com/watch?v=DlP3FZbAXgI"
	APIKEY="?"
	
	$ curl --request POST \
     --url https://www.virustotal.com/api/v3/urls \
     --header 'accept: application/json' \
     --header 'content-type: application/x-www-form-urlencoded' \
     --header 'x-apikey: ${APIKEY}' \
     --data 'url=${URL}'
	```


#### 6. Verificar Anexos

- Analise os anexos para detectar possíveis malwares:
    - use sandbox ou ferramentas de análise (ex.: [Cuckoo Sandbox](https://cuckoosandbox.org/about.html)).
- Bloqueie arquivos executáveis (.exe, .js, .bat)
	- mesmo que estejam compactados.

- Para essa tarefa, podemos usar a ferramenta de análise [email-analyses](https://github.com/keraattin/EmailAnalyzer)
	- procurando por arquivos 
	```bash
	$ python3 EmailAnalyzer/email-analyzer.py -a -f header_bichodepe.eml 
		
	 █████╗ ████████╗████████╗ █████╗  ██████╗██╗  ██╗███████╗
	██╔══██╗╚══██╔══╝╚══██╔══╝██╔══██╗██╔════╝██║  ██║██╔════╝
	███████║   ██║      ██║   ███████║██║     ███████║███████╗
	██╔══██║   ██║      ██║   ██╔══██║██║     ██╔══██║╚════██║
	██║  ██║   ██║      ██║   ██║  ██║╚██████╗██║  ██║███████║
	╚═╝  ╚═╝   ╚═╝      ╚═╝   ╚═╝  ╚═╝ ╚═════╝╚═╝  ╚═╝╚══════╝
	                                                          
	
	[1]->Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf
	
	```
	- procurando arquivos anexos e analisando-o quando encontrado.
	```bash
	$ python3 EmailAnalyzer/email-analyzer.py -f header_bichodepe.eml --attachments --investigate  
	
	[1]->Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf
	 █████╗ ███╗   ██╗ █████╗ ██╗  ██╗   ██╗███████╗██╗███████╗
	██╔══██╗████╗  ██║██╔══██╗██║  ╚██╗ ██╔╝██╔════╝██║██╔════╝
	███████║██╔██╗ ██║███████║██║   ╚████╔╝ ███████╗██║███████╗
	██╔══██║██║╚██╗██║██╔══██║██║    ╚██╔╝  ╚════██║██║╚════██║
	██║  ██║██║ ╚████║██║  ██║███████╗██║   ███████║██║███████║
	╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝╚══════╝╚═╝   ╚══════╝╚═╝╚══════╝
	
	- Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf
	
	Virustotal:
	[Name Search]->https://www.virustotal.com/gui/search/Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf
	[MD5]->https://www.virustotal.com/gui/search/d41d8cd98f00b204e9800998ecf8427e
	[SHA1]->https://www.virustotal.com/gui/search/da39a3ee5e6b4b0d3255bfef95601890afd80709
	[SHA256]->https://www.virustotal.com/gui/search/e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
	
	```

- Verificando o arquivo PDF
	- com a tool [PDFid](https://gitlab.com/kalilinux/packages/pdfid)
		```bash
		$ python pdfid.py "Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf"
		PDFiD 0.2.7 Anjali Khatri, Vikram Khatri - Mastering Service Mesh_ Enhance, secure, and observe cloud-native applications with Istio, Linkerd, and Consul-Packt Publishing (2020).pdf
		 PDF Header: %PDF-1.4
		 obj                 8562
		 endobj              8562
		 stream               934
		 endstream            934
		 xref                   2
		 trailer                2
		 startxref              2
		 /Page                611
		 /Encrypt               0
		 /ObjStm                0
		 /JS                    0
		 /JavaScript            0
		 /AA                    0
		 /OpenAction            2
		 /AcroForm              1
		 /JBIG2Decode           0
		 /RichMedia             0
		 /Launch                0
		 /EmbeddedFile          0
		 /XFA                   0
		 /Colors > 2^24         0
		```


#### 7. Monitorar Comportamento

1. Registre todas as interações relacionadas ao e-mail (ex.: cliques em links).
2. Use logs de servidores e ferramentas de monitoramento para identificar atividades suspeitas.

#### 8. Indicadores suspeitos

- **Falhas em SPF, DKIM ou DMARC**:
    - IP não autorizado, assinaturas inválidas ou falha no alinhamento DMARC.
- **Domínios suspeitos**:
    - Domínios recém-registrados ou inconsistentes com o remetente.
- **URLs encurtadas ou mascaradas**:
    - Ex.: `bit.ly/suspeito`.
- **Linguagem e gramática incomuns**:
    - Erros ortográficos ou solicitações urgentes e atípicas.
- **Anexos inesperados**:
    - Especialmente se solicitarem execução.
    - Arquivos do Microsoft Office
    - PDFs
    - .ZIP e .RAR
    - .ISO e .IMG

#### 9. Conclusão

Se seguir esses passos, você terá uma abordagem estruturada para detectar e-mails maliciosos e mitigar riscos analisando e-mails utilizando padrões como **SPF**, **DKIM** e **DMARC** é essencial para identificar fraudes e prevenir incidentes de segurança e o processo deve ser complementado com ferramentas de análise automatizada e uma política de conscientização para usuários. 

---

[...em breve melhorias no how to]

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

