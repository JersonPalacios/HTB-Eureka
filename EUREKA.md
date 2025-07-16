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
   Dirección IP: `10.10.11.66`

   

