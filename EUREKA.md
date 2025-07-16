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



   



   

