# Internetwache CTF 2016 : Oh Bob!

**Category:** Crypto
**Points:** 60
**Solves:** 167
**Description:**

> Description: Alice wants to send Bob a confidential message. They both remember the crypto lecture about RSA. So Bob uses openssl to create key pairs. Finally, Alice encrypts the message with Bob’s public keys and sends it to Bob. Clever Eve was able to intercept it. Can you help Eve to decrypt the message?
> Attachment: [crypto60.zip](./crypto60.zip)

## Write-up

We are given 3 RSA public keys (bob.pub, bob2.pub, bob3.pub), and 3 secrets encrypted with openssl in secret.enc.
Our goal is to decrypt them to get the flag.

We first extracted the modulus and exponent from each of the 3 public keys using the following command
```sh
$ openssl rsa -pubin -inform PEM -text -noout < bob.pub
Modulus (228 bit):
    0d:56:4b:97:8f:9d:23:35:04:95:8e:ed:8b:74:43:
    73:28:1e:d1:41:8b:29:f1:ec:fa:80:93:d8:cf
Exponent: 65537 (0x10001)
```
(Thanks to stackoverflow and the answer to this question: http://stackoverflow.com/questions/3116907/rsa-get-exponent-and-modulus-given-a-public-key)

After repeating this for all keys and converting to decimal, we got the following 3 public key pairs:

bob.pub
```
n = 359567260516027240236814314071842368703501656647819140843316303878351
e = 65537
```
bob2.pub
```
n = 273308045849724059815624389388987562744527435578575831038939266472921
e = 65537

bob3.pub
```
n = 333146335555060589623326457744716213139646991731493272747695074955549
e = 65537
```

The moduli are relatively small, and can be factored in a few minutes using the Quadratic Sieve or GNFS algorithms (we used the Msieve program).
From p and q, we can then find phi(n) = (p-1)*(q-1) and the private exponent d = e^(-1) mod phi(n). We used WolframAlpha for the computations.
We now theoretically have all necessary data to decrypt the message, but our knowledge of the openssl file formats was somewhat limited
so we had to look around until we found this helpful tutorial on manually building RSA private keys from the parameters:
http://stackoverflow.com/questions/19850283/how-to-generate-rsa-keys-using-specific-input-numbers-in-openssl

From this we see that some additional data is required (openssl probably uses the Chinese Remainder Theorem optimization for RSA),
which we calculate as well with WolframAlpha. After all the parameters were calculated and stored in the right format, in the following files:
[private1](./private1)
[private2](./private2)
[private3](./private3)

The binary DER files were then generated using
```sh
$ openssl asn1parse -genconf private1 -out key1.der
    0:d=0  hl=3 l= 154 cons: SEQUENCE          
    3:d=1  hl=2 l=   1 prim: INTEGER           :00
    6:d=1  hl=2 l=  29 prim: INTEGER           :0D564B978F9D233504958EED8B744373281ED1418B29F1ECFA8093D8CF
   37:d=1  hl=2 l=   3 prim: INTEGER           :010001
   42:d=1  hl=2 l=  29 prim: INTEGER           :010266F631885301D037017A38F3B3196B3491CE1C97007055FD220001
   73:d=1  hl=2 l=  15 prim: INTEGER           :0375AC9161AD7E431EBDDF01E514CF
   90:d=1  hl=2 l=  15 prim: INTEGER           :03DAE2E1D28965D328B06D615DFC01
  107:d=1  hl=2 l=  15 prim: INTEGER           :01AFC69781B5210EFBD7B8F6A585C5
  124:d=1  hl=2 l=  14 prim: INTEGER           :105E58FE83F6E364B6606A100401
  140:d=1  hl=2 l=  15 prim: INTEGER           :CD8DEAFC5A1B933414DA0A5CCA95
```
to obtain the files
[key1.der](./key1.der)
[key2.der](./key2.der)
[key3.der](./key3.der)

The secrets are finally decrypted using the following command:
```sh
$ echo "DK9dt2MTybMqRz/N2RUMq2qauvqFIOnQ89mLjXY=" | base64 -D | openssl rsautl -keyform DER -inkey key1.der -decrypt
IW{WEAK_R

$ echo "CiLSeTUCCKkyNf8NVnifGKKS2FJ7VnWKnEdygXY=" | base64 -D | openssl rsautl -keyform DER -inkey key2.der -decrypt
SA_K3YS_4R

$ echo "AK/WPYsK5ECFsupuW98bCFKYUApgrQ6LTcm3KxY=" | base64 -D | openssl rsautl -keyform DER -inkey key3.der -decrypt
3_SO_BAD!}

```
(Notice that the order of the last two messages is inverted. The 3rd message was encrypted with key2, and the 2nd with key3).

After combining the 3 messages, we get the flag:
>  IW{WEAK_RSA_K3YS_4R3_SO_BAD!}

# CTF-Writeups
