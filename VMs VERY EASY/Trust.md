VM Trust

Realizamos un escaneo de puertos, servicios y versiones con el siguiente comando en NMAP:

```bash
nmap -sS --open -vvv -Pn 172.18.0.2
```
Resultado: 


<img width="816" height="355" alt="Captura de pantalla 2025-06-02 190113" src="https://github.com/user-attachments/assets/cc3d294b-53c5-416d-8542-63f47bf38724" />

Obtenemos 2 puertos abiertos: 22 y 80
Seguidamente realizamos un escaneo con NMAP para saber sus versiones:

```
nmap -sV -p 22,80 -vvv 172.18.0.2
```
