
# UofTCTF 2024

I'll break down the steps to solve **Illusion - Forensics** challenge.

# Challenge Description

Good Luck.

## Solution

We are provided with a file called `traffic.pcapng`.
After exploring the file for a while, we can notice an unusual thing in the **http** traffic, which is the **GET HTTP requests**. it contains a weird path and parameter called `/images?guid=` with a **base64** encoded data.
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Illusion/Screenshots/Screenshot1.png">

Decoding the base64 data will not give us anything readable. The data appears to be encrypted.
I exported the files from the http traffic.
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Illusion/Screenshots/Screenshot2.png">

and then viewed one of the files and it looks like google webpage
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Illusion/Screenshots/Screenshot3.png">

After some searching, I found out that it looks like [TrevorC2](https://github.com/trustedsec/trevorc2) traffic.
This is from the TrevorC2 repo:
>TrevorC2 is a client/server model for masking command and control through a normally browsable website. Detection becomes much harder as time intervals are different and does not use POST requests for data exfil.

I read this article about **TrevorC2** and it was very useful : https://nasbench.medium.com/understanding-detecting-c2-frameworks-trevorc2-2a9ce6f1f425

So now we know that the C2 conversation is hidden in two things:

 1. The server side hides its commands in `<!-- oldcss=>` 
 2. The client side hides its data in `/images?guid=`
 
 Let's extract the client side data. you can use tshark to extract the data:

    tshark -r traffic.pcapng -Y "http.request" -T fields -e http.request.uri > clientdata.txt
Now we can decrypt the data, to decrypt the data we can check the **TrevorC2** repo to see the decryption method.
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Illusion/Screenshots/Screenshot4.png">
We still need the encryption key :( , Lets try the default one which is `Tr3v0rC2R0x@nd1s@w350m3#TrevorForget` to see if it works.
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Illusion/Screenshots/Screenshot5.png">

    ct=["VWZvY0RaWTd1a2kwYmFkMlhyYXpBRWFNRVhNbTZwUFgrUUtVenhKdWtrUDlib3p3cUNIS2Y2S3owNGVsRVdvZw==","ajcrT1pmTC9kMy8xNkRXQzFvdStUWFo5ZVV1OTgyMDduV0R1dWJoemdRUTN5OXFIQ2NXY2VKOFhKM3FQRGRKQldIVkgzYW1RYkZ5ckdHNVZtOGs4SXNkZEdpaHhVeGR2aFdYN3dLMUF6RjZOM1FRc2trYkdjZFYrb1RLR3ZPb2puQUEzTUpLakRQd1FlREg1ZlFZdmp3PT0=","WkN0eTk1TGdFb1JrZWZIL29VU0hhV0t0VjJocmJnTjVDUDR4eWU1OU8vSEppeVp6QnJqS1IvcUprOVZPUVpHYnk2RjArZFQ0K25BTUkreEFtM2xxK00zVEZyQk1DQ3k4MFUwNHVaSCszSUVCaDBrZ3ZDVWo4Tk0wMGNQblNZdDlqOTVBVzRCelBNYXV4cE9wbUViamN4N042QzBPckhNcVFtR2MybjBOTVh6Q0NmZlAzWXQ4dThkN0ozeWtYK0hEZmkrSTdQTmdBRlA2RTJKblRNVGdZOGVVUGU3enJmYzVEb0RydG9ETVhWZ2dPMElZQUF3b3pDUEN3eUZTdmxHeU5iV3VpYVpaSVR2TVNnOW9leW9RZU10bVBRZXQ2ejV4Zm4rZDQ0eHFNdkhRR1llNXJZVERQSzYwYUJUbDMrNmRhUUlzbXRieXpwaW1nQjlQWmV1dVdDUytYVmhQNXlCcHh1cjJ3L1BZTzRsLzVFTFFYa3JUblF3eTRxc25yMlR4YW9idk1LN3FzaGtFVGZsbkZwMktxY09SUTZMTlo4bEVUOTdxQ2tHaDNkTGdjTVFMc2E0M3ZLdVp4YzdBOUcwOFFRUjBjMHlLK3ptaVNnUCtjM2djeEg3UzVweU1NNlJNakd5Nk9neHcvbTB0QTJEcjJRWDBBTU9YRm53MEpYbzhNcTVwWUhndGw2R0laak5OcUtWdG5JcnVpb0RTcW1iSmF1T295bngrbjNNdjVrWVZ1QXVQbDB4YXU1Q2ZOZERrWnRJaVZZUW1JZmpsZU1CaHM1emRKWVREVXhhK2JwOXB3UHkxR21UZW4rK2QwZks4VGFEdXB3eWdqSnRTc1pydXV6eDZ3RVliWG95dnUrQlhHcFE5Y1NCRWpnN2FmcWxzbGtmbDJ4R1FTWi9Sdy96ZGxKeFA4SEZKc3oweE5PTC9CV3QzVUVMLzViZjljQ1Vva2tETFFQV2hHajJPZmYvU2xoU0o0eUx4TCtNPQ==","UzltSmsxK2RQM0ROWWJoWGwyWC9mSnZBY1VkSkJVeUVDS0hzNXZMOXoxYz0=","ZTcreEtlMzFSamMrRGNsMVloWDJyVmJuTFJIdzY5VGFZZWNqbzlxb1IvcjVsQnpIMU1YS2YrbFZEZWdiQzM1M3I0QVpqSUY5UHF3ck15LzF4R3ZjLzFINWRxeFdWRVk4NWdLclUxM0VBeEM1U3lxSU8xMnlrRkpNUG1XRTFNWUM=","ME9BMzRkK1I0cFAwc2pmTEFzWFk0N3lPMDBFRjZ1bGVGUnc0RGxlZ1RRdz0=","S2VHSVF1RGVhMjFrTm5vM1hoR052U25yVzZnSy9sWCtnYVN1WTVJSnJSRitmd20xU2x3K2xCeXVsZUNWS3VCUkJxTUJQYjRQWERmV1Zjb29mbUtqRHFSbFU4alZyTzZ2eVhxbkdsdXlINCtxZ0NQQWN2bFBzeUxGbEJOOWk2L2FlS0hhQmJqR0lqL1FPalJWbHFnL21ZRnY2SG1uUGMrK3o3KytKcGdpTlNoQWtJZzM1QzZaZ1VXTXg1ck9qNmduTmQxaDRFTzcxME9PakpnbEpsbVFoOHdSUWpKWkh0cnA1N3ovOUgrblJHR2FZeHp3c1F6YlJxSWx3WVZpOGVmNlBlRlJYMFVIR0h3MittMGkxS1ErQ2lHUlB6ekhhYmIxSDlxRE50cHZMck5hNWhPV1hYblJEVHc1RjQwQTBEUkhVQXV1OWcwT1NZTExtNGk3MGFaWmVUd1FhYk9XQjRucTVIR2hNcHBVS2hEb0V1YnhlMHM3Z2VuR2loVmtaRUo0NVpkYTdnNEFLWjh2L1pjdUxHYkZGZjJhdzNoS3NBemZ3R09wc0JxblA4WTh6amRZbERlQmpqWVMxRkd3NHNOMEpPMWdBeTl4TmI4NlBlYXJORVRBdFlJdGVnaGZDUnpBcHVnODhnR0dHa3h6UVZEakdUbkg3K3VUWWMyMkJQM2I3LzNTNlQxc29KbzBJbGxZbEdwQ2srWHh4QjN5Z0QzbXNicnUzekJuazlDWTh2MFVYcEl4V2FTZUlmWXlGT3Jxa1pqbkoxWTNRU1hGcUhBRmNwNUc5dz09","OVdReUJzZ0MwR3JNRGljNnNqaEdPYzB5b1dTUUpwMkJMcGxzTyszSWJUUT0="]
    
    import hashlib
    import base64
    from Crypto.Cipher import AES
    class AESCipher(object):
        """
        A classical AES Cipher. Can use any size of data and any size of password thanks to padding.
        Also ensure the coherence and the type of the data with a unicode to byte converter.
        """
        def __init__(self, key):
            self.bs = 16
            self.key = hashlib.sha256(AESCipher.str_to_bytes(key)).digest()
    
        @staticmethod
        def str_to_bytes(data):
            u_type = type(b''.decode('utf8'))
            if isinstance(data, u_type):
                return data.encode('utf8')
            return data
    
        def _pad(self, s):
            return s + (self.bs - len(s) % self.bs) * AESCipher.str_to_bytes(chr(self.bs - len(s) % self.bs))
    
        @staticmethod
        def _unpad(s):
            return s[:-ord(s[len(s)-1:])]
    
        def encrypt(self, raw):
            raw = self._pad(AESCipher.str_to_bytes(raw))
            iv = Random.new().read(AES.block_size)
            cipher = AES.new(self.key, AES.MODE_CBC, iv)
            return base64.b64encode(iv + cipher.encrypt(raw)).decode('utf-8')
    
        def decrypt(self, enc):
            enc = base64.b64decode(enc)
            iv = enc[:AES.block_size]
            cipher = AES.new(self.key, AES.MODE_CBC, iv)
            return self._unpad(cipher.decrypt(enc[AES.block_size:])).decode('utf-8')
    
    # add cipher key here
    CIPHER = ("Tr3v0rC2R0x@nd1s@w350m3#TrevorForget")
    cipher = AESCipher(key=CIPHER)
    
    
    for i in ct: 
        print(cipher.decrypt(base64.b64decode(i)))
                
It worked !
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Illusion/Screenshots/Screenshot6.png">

We've gotten one side of the conversation. Now we need the other part which is the server side.
To extract the **oldcss** data, I simply used the OG method of `strings` and `grep` and I parsed them manually 

    strings traffic.pcapng | grep "oldcss"
Lets feed the data again to our script 

    ct=["cFl2gZ64Xe9FdCr2nMGlPPHKr//oqB/bWOaDiLiOLmQ=","KOw3LLlCqO1H0/OzTaSCtdtDQEc1kaUHYDHK3URMinE=","kofJyYXCyx+kuQkUISD8xbS+3wGEninpPGkCfO7cSdk=","pthScpHMvLajfJArnoBOuaJE4qyruZUvV9RH1W1xJglZllVYTUApMAqfCX/Fe0hvq8Ym5FBzjg6aIGGOO5HTBA==","cLLXq0nk9MPX9fcJgK7PdYhjHwrGu6qEkLPMNV7+QQ4=","uMS9CwMZkXrXyXA4LTbRKL6FJkKBxyM9Pi96rGVuHEGyu1h+Qa7UIPej/b2PH1QL","sGpByUYockKmYpDyu84eFIOWENercq7Twup5JVfoh6w=","3unUUEgRnp/Cb5zCdqtNZk10QW1jno8SJ1x5elH408U=","TJMEJ6awBY5OJO8R/DuIj1oQNItQ6QvaBkpb0vsGT6Cmj2DOIIhgAkVOw6M9xrey","IkMrNxaUs4zRtjU81TvQAaLFGe2L9wp3lowIDxexlEg=","seiTsoy2I3arpIXH3DhPqLWDq3Q5VyFfTSPg4FLgJnwzBNJwnFKoQmpit2Bihe7+","noJvVRGs/silHFz2oPTldiey+XvgYNsutIOo4xVi2i4=","C9XqWpYeqCIn8Dk8gCVtpdg47vm8e8peFqkfQJ6WVbUvL7ucvQ0ayWnKRBF2GI+ltFBWNMa+wawqeuvFK61RGvKVWogAqAVg4J7qmScn+HRF0QZFgEunXlAduM+16nnf","0uVBecny+wd0QC7IC2UV0O/5itgE4ZMSRKBkaeNvLbM="]
    
    import hashlib
    import base64
    from Crypto.Cipher import AES
    class AESCipher(object):
        """
        A classical AES Cipher. Can use any size of data and any size of password thanks to padding.
        Also ensure the coherence and the type of the data with a unicode to byte converter.
        """
        def __init__(self, key):
            self.bs = 16
            self.key = hashlib.sha256(AESCipher.str_to_bytes(key)).digest()
    
        @staticmethod
        def str_to_bytes(data):
            u_type = type(b''.decode('utf8'))
            if isinstance(data, u_type):
                return data.encode('utf8')
            return data
    
        def _pad(self, s):
            return s + (self.bs - len(s) % self.bs) * AESCipher.str_to_bytes(chr(self.bs - len(s) % self.bs))
    
        @staticmethod
        def _unpad(s):
            return s[:-ord(s[len(s)-1:])]
    
        def encrypt(self, raw):
            raw = self._pad(AESCipher.str_to_bytes(raw))
            iv = Random.new().read(AES.block_size)
            cipher = AES.new(self.key, AES.MODE_CBC, iv)
            return base64.b64encode(iv + cipher.encrypt(raw)).decode('utf-8')
    
        def decrypt(self, enc):
            enc = base64.b64decode(enc)
            iv = enc[:AES.block_size]
            cipher = AES.new(self.key, AES.MODE_CBC, iv)
            return self._unpad(cipher.decrypt(enc[AES.block_size:])).decode('utf-8')
    
    # add cipher key here
    CIPHER = ("Tr3v0rC2R0x@nd1s@w350m3#TrevorForget")
    cipher = AESCipher(key=CIPHER)
    
    
    for i in ct: 
        print(cipher.decrypt(i))
        
        
<img src="https://github.com/HashemSalhi/CTF-Writeups/blob/main/UofTCTF%202024/Forensics/Illusion/Screenshots/Screenshot8.png">

**uoftctf{Tr3V0r_C2_1s_H4rd_T0_D3t3c7}**
