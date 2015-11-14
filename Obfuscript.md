##Obfuscated (400)

#Problem

How good are you at reading Python? Put your Python skills to the test by finding the password that passes this check: obfuscated.py
Note: easyctf{} formatting is not required for this problem.
#Hint

Note: there may be multiple correct answers. The grader runs the same code that's in the given file.

#Solution
OK. So reading this program in Gedit, you see it is:
```python
from string import ascii_uppercase as v, ascii_lowercase as k
def check_flag(s):
	if len(s) != 0x19:
		return False
	s = list(s)
	if int(`[s.pop(r) for r in ([2] + [i for i in range(11, 18, 2)] + [20])[::-1]][::-1]`[2::5]) != 0x61a83:
		return False
	if len(list(set([s.pop(r) for r in map(lambda p: int(p, 2), [("1"*5)[:2], ("1"*5)[2:]])[::-1]]))) != s.index("h"):
		return False
	y, z = [], []
	u = len(list(set([repr(y.append(s.pop(-1))) + repr(z.append(s.pop(-1))) for w in range(2)]))) - 1
	if u != len(list(set(y))) ^ len(list(set(z))):
		return False
	if (ord(y[u]) ^ ord(z[u])) ^ 0x1e != 0:
		return False
	if v.index(s.pop()) ^ len(s) ^ 0x1e != 0:
		return False
	a, i = filter(lambda c: c in v, s), filter(lambda c: c in k, s)
	if map(lambda a: a + 0x50, [7, 2, 4, -8]) + [0x4f] * 4 != map(ord, a):
		return False
	i[1:3] = i[2:0:-1]
	if i != list("hate"):
		return False
	return True
```
It is quite clear that whatever string "s" is being input, it must pass all of the tests to return true. So we can rewrite the script to be:
```python
from string import ascii_uppercase as v, ascii_lowercase as k
if len(s) != 0x19: 
	print "Rekt"
s = list(s)
if int(`[s.pop(r) for r in ([2] + [i for i in range(11, 18, 2)] + [20])[::-1]][::-1]`[2::5]) != 0x61a83:
	print "Fail1"
if len(list(set([s.pop(r) for r in map(lambda p: int(p, 2), [("1"*5)[:2], ("1"*5)[2:]])[::-1]]))) != s.index("h"):
	print "Fail2"
y, z = [], []
u = len(list(set([repr(y.append(s.pop(-1))) + repr(z.append(s.pop(-1))) for w in range(2)]))) - 1
if u != len(list(set(y))) ^ len(list(set(z))):
	print "Fail3"
if (ord(y[u]) ^ ord(z[u])) ^ 0x1e != 0:
	print "Fail4"
if v.index(s.pop()) ^ len(s) ^ 0x1e != 0:
	print "Fail5"
a, i = filter(lambda c: c in v, s), filter(lambda c: c in k, s)
if map(lambda a: a + 0x50, [7, 2, 4, -8]) + [0x4f] * 4 != map(ord, a):
	print "Fail6"
i[1:3] = i[2:0:-1]
if i != list("hate"):
	print "Fail7"
```
If the program returns `NULL`, then we have suceeded. So the first test compares the length of input to `0x19`, which is hex for 25. So the string length is 25. In order to be able to run the tests and effectively detirmine what the string is, we can make the string `abcdefghijklmnopqrstuvwxy`, representing the 25 placeholders.

The second test is a long, confusing function. While it is possible to work out, writing a second script to just
```python
s="abcdefghijklmnopqrstuvwxy"
s=list(s)
print (`[s.pop(r) for r in ([2] + [i for i in range(11, 18, 2)] + [20])[::-1]][::-1]`[2::5])
print 0x61a83 
```
This returns `clnpru` and `400003`. This means that the `int(of whatever)=clnpru` when it should be `int(of whatever=400003`
So we replace the letters c l n p r u with 4 0 0 0 0 3 respectively to get the string `ab4defghijk0m0o0q0st3vwxy` , and if runs through the test will pass the second category.
The next gate can also be tested by adding: 
```python
print list(set([s.pop(r) for r in map(lambda p: int(p, 2), [("1"*5)[:2], ("1"*5)[2:]])[::-1]]))
print s
print s.index("h")
```
to the code. It is important to print the list s after popping to see how it has dynamically shifted.
This yields the output:
```python
[i,e]
['a', 'b', 'd', 'f', 'g', 'h', 'j', 'k', 'm', 'o', 'q', 's', 't', 'v', 'w', 'x', 'y'] 
5
```
The S index function calls the first time that the character h appears in the list,
this means that the `len()` of the first output, which is the variable called in the if statement must be equal to the first time the h appears in the NEW LIST
By substituting in h for d, the if statement is passed over because `2==2`
So we have found the new string to be `ab4hefghijk0m0o0q0st3vwxy`
The next gate is where it gets tough. It defines 2 arrays and a variable u, which when printed is `0`
Conviniently, the second part of the if statement also returns `0`, so this gate is passed automatically.

The next gate is where it actually gets tough. First it is important to note that `^` means XOR (https://en.wikipedia.org/wiki/Bitwise_operation#XOR). So first, printing the contents of Y and Z reveals it to be
```python
['y', 'w']                                                                                                                 
['x', 'v'] 
```
Thereore `y[0]` and `z[0]` are y and x respectively. The `ord` then takes the ASCII value of said characters. So the binary bits of the ASCII's of each character should equal the binary of `30`, which is `0b11110`.
We did not know much about binary, so a guess and check method was effective. By printing the `bin()` of each of the ASCII values, which for y and x turned out to be `0b1111001` and `0b1111000` respectively. This is where we utilized a bit of guess and check. By plugging in integers between 0-9 into where y and x previously were. Various patterns emerged, but the important one was as the integer in the x area decreased by 1, if it was even, the result would be an integer that was the sum of the two. If it was odd it was the difference of the x area and the y area. When we tried various combinations of integers, it was clear no obvious solution was present. So we switched over to characters, the first combination being `a` and `z` which returned `27`. So we tried using the integer logic when decreasing the letter a by one character, and sure enough the combination `d` and `z` returned `30` which when XORed with `0x1e`, returns `0`, passing the gate. Therefore our new string is `s="ab4hefghijk0m0o0q0st3vwdz"`

The next gate is:
```python
if v.index(s.pop()) ^ len(s) ^ 0x1e != 0:
	print "Fail5"
```
It is saying that the value popped (the value at the end of the list, which when printed is `t`) the first iteration of said value in the list of capital letters XORed with the `len(s)` which is `12`, should be equal to `0x1e` or `30`. Since the 12 is constant, we can say that `?^12==30`, quick expirimentation gives this value to be `18`, which would be the 19th letter in the alphabet, or `S`. Our new string is now `s="ab4hefghijk0m0o0q0sS3vwdz"`
The next gate defines `a, i = filter(lambda c: c in v, s), filter(lambda c: c in k, s)`. `a` all of the upperase letters in `s`, and i is all the lowercase. Keep in mind this is not of the original string, but of the dynamic one, which when printed is: `['a', 'b', 'h', 'f', 'g', 'h', 'j', 'k', 'm', 'o', 'q', 's']`. The printing both contents of the if statement reveals them to be:
```python
[87, 82, 84, 72, 79, 79, 79, 79]                               
[] 
```
Looking at the contents of the if statement it is shown the `ord()` of the contents of `a` must be equal to the first output. This is easy, just replace the first eight letters in the new list with that of the `chr()` of those ASCII values. We avoided replacing anything that was being used in the previous gates. Our new string was: `s="WR4heTHOiOO0O0o0q0sS3vwdz"`

OK. THE LAST GATE. It defines `i[1:3] = i[2:0:-1]` and states:
```python
if i != list("hate"):
	print "Fail7"
```
Printing i reveals it to be `['h', 'q', 'o', 's']` and it should equal `['h', 'a', 't', 'e']`. Well simply just modify the string to finally get `s="s="WR4heTHOiOO0O0t0a0eS3vwdz"`.

flag=WR4heTHOiOO0O0t0a0eS3vwdz
