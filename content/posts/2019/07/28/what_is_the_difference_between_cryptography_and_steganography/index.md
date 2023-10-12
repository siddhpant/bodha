---
title: What is the difference between cryptography and steganography?
date: 2019-07-28
tags: ["cryptography", "steganography", "migrated", "quora"]
---

{{< figure src="index.assets/cryptography.jpg" caption="Source: https://www.slideshare.net/kinleay/cryptography-its-history-application-and-beyond" >}}

Cryptography involves itself with encryption/decryption or hashing of the messages. ***It's almost a mathematical field which has a profound practical impact on our lives.*** Generally, other guys must not be able to read your data, even if they get hold of it and know how it was encrypted.[^kerckhoff]

[^kerckhoff]: [Kerckhoffs's principle - Wikipedia](https://en.m.wikipedia.org/wiki/Kerckhoffs%27s_principle)

*Some of the most noticeable everyday life examples:*

-   Most n00b example everyone comes across:
    -   Fb lbh svanyyl qrpelcgrq guvf! :-C
    -   If you didn't know words are shifted by 13, this would be garbage for you.
-   To check if data is untampered as it was originally meant to be.[^checksum]
-   It's used to transmit data/messages in a secure and safe way so that any  interceptor or evesdropper doesn't get hold of what we are communicating or sending, i.e. secrecy of transmission of communication or data. One  way to implement this is the RSA algorithm. But it's not the only way and a myriad of algorithms, more or less powerful, exist.
    -   The lock you see on HTTPS websites is a direct consequence of this.[^public-key]
-   As a related point to above, the infamous enigma machine.[^enigma]
-   Storage and verification of passwords while logging in or otherwise.
-   Encryption of your stored data.[^storage-encryption]
    -   Having a lock/full disk encryption on your mobile. Your Android phone is decrypted on every reboot.[^fde]
-   *Steganography* (see the next section).

[^checksum]: [Checksum - Wikipedia](https://en.m.wikipedia.org/wiki/Checksum)
[^public-key]: [Public key certificate - Wikipedia](https://en.m.wikipedia.org/wiki/Public_key_certificate)
[^enigma]: [Enigma machine - Wikipedia](https://en.m.wikipedia.org/wiki/Enigma_machine)
[^storage-encryption]: [What is Storage Encryption?](https://www.thalesesecurity.com/faq/encryption/what-storage-encryption)
[^fde]: [Full-Disk Encryption  |  Android Open Source Project](https://source.android.com/security/encryption/full-disk)

And much much much much more.

Steganography involves itself with hiding some file (image/executable/any file or data) ***in another file.*** Essentially you have data hidden in data, like, you hide a paper in a big folder.  Not much mathematical theory (when compared to cryptography) is involved in this and is many times supplemented by cryptography.

It's advantage over cryptography is that since data is hidden ***there's a far less probability of suspicion than some exposed cryptic data in case of vanilla cryptographic data transfer.*** For example, let Germans use the enigma machine and suppose British got hold of it accidentally with a blank mind. Seeing the enigma message  would immediately ring bells since the cryptic sequence seems too good  to be random. Whereas in steganography that would be hidden behind  another thing, like suppose a dank meme on Hitler and Churchill.

*Some normal uses in a huge set of uses:*

-   Very very very 5yo tier n00b rudimentary and a bad practical example is:
    -   NwnyynynfoofynyncNfwnxppbatengfnyncjyjxrwxfjxfykznxOmfwfaqhcibgrgurnafjrenxnyynfzfafxwnxuffygunajfzana
    -   Find a message in this! :)
-   Someone can hide an executable code in an image. Classic virus propagation strategy.[^png-payload]
-   You can hide your files behind another file.
-   Hide a message in an image so that it can be seen in a particular colour value only.[^wikipedia-steg] Or in audio files.[^wav-steg] List is endless.
-   Printers printing barely visible data on every page about the machine and date time, so that the document can be traced back.[^printer-mic]

[^png-payload]: [PNG Embedded – Malicious payload hidden in a PNG file](https://securelist.com/png-embedded-malicious-payload-hidden-in-a-png-file/74297/)
[^wikipedia-steg]: [File:Steganography.png - Wikipedia](https://en.wikipedia.org/wiki/File:Steganography.png%23mw-jump-to-license)
[^wav-steg]: [Steganography and .WAV](https://bastijn.io/2018/Steganography-and-WAV.html)
[^printer-mic]: [Machine Identification Code - Wikipedia](https://en.m.wikipedia.org/wiki/Machine_Identification_Code)

And much much much more.

Thanks for reading!
