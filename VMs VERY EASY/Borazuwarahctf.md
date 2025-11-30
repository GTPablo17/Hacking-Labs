VM borazuwarahctf

Realizamos un escaneo de puertos, servicios y versiones con el siguiente comando en NMAP:

```
nmap -sV  --open -Pn 172.17.0.2
```

Resultado:

<img width="791" height="230" alt="Pasted image 20250610094819" src="https://github.com/user-attachments/assets/d3b81bd9-f77d-4739-8551-c02b1a022cd0" />


Tenemos 2 puertos abiertos, procedemos a ver la pagina de Firefox:

<img width="665" height="596" alt="Pasted image 20250610095003" src="https://github.com/user-attachments/assets/60e67c24-423a-4c82-a4e6-15589e9342f9" />

Vemos una imagen de Kinder sorpresa, en el codigo de fuente de la pagina no vemos nada, procedemos a usar GOBUSTER:

<img width="999" height="721" alt="Pasted image 20250610100305" src="https://github.com/user-attachments/assets/f55ef63f-9f4c-4f43-9d38-892a3bf60f00" />

Como no encontramos nada con Gobuster descargamos la imagen y la analizamos con ESTEGANOGRAFIA utilizando Steghide:

```
steghide extract -sf Untitled.jpg
```

Resultado:

<img width="450" height="76" alt="Pasted image 20250610102831" src="https://github.com/user-attachments/assets/40b89d41-67f0-46a1-be37-fbc8ae027f2f" />

Cuando lo aplicamos nos pedirá una passphrase como no tenemos le damos a enter y vemos que realmente si que había un archivo oculto llamado `secreto.txt`. el mensaje: `wrote extracted data to "secreto.txt".` indica qeu se extrajo correctamente, ahora lo vemos con el comando `cat`:
