# 🛡️ Ataque DHCP Spoofing

📹 [Video demostración](https://youtu.be/KckrnfweW6I) 

---

## 📌 Objetivo del Laboratorio

Demostrar cómo un servidor DHCP falso puede responder a las solicitudes DHCP de los clientes antes que el servidor legítimo, permitiendo observar el impacto que tiene la asignación de configuraciones de red controladas por un atacante.

### Topología de Red
![Topología de Red](https://res.cloudinary.com/drzcveg06/image/upload/v1780767919/DSPOOFING-1_ef4sjw.png)

---

## 🎯 Objetivo del Script

Crear un *servidor DHCP falso* capaz de responder a solicitudes DHCP Discover y DHCP Request, asignando direcciones IP, gateway y servidores DNS definidos por el atacante con el fin de *redirigir el tráfico* de los clientes de la red.

---

## ⚙️ Requisitos

### Hardware y Software

- Un equipo que actuará como *atacante* (Kali Linux)
- Un *servidor DHCP legítimo* presente en la red
- Al menos un *cliente* configurado para obtener IP automáticamente vía DHCP
- Todos los dispositivos en la *misma red local*
- *Python 3* instalado
- Biblioteca *Scapy* instalada
- Permisos de *administrador (root)* para ejecutar el script
- Una interfaz de red con acceso al segmento donde están los clientes DHCP

### Instalación de dependencias

bash
pip install scapy


---

## 🔧 Parámetros Utilizados

| Parámetro | Descripción |
|-----------|-------------|
| Interfaz de red | Adaptador donde se ejecutará el servidor DHCP falso |
| IP del servidor falso | Dirección IP que identificará al servidor DHCP falso |
| Gateway falso | Dirección IP de puerta de enlace entregada a los clientes |
| DNS falso | Dirección IP del servidor DNS anunciada a los clientes |
| Máscara de subred | Máscara de red asignada a los clientes |
| Primera IP del pool | Dirección inicial del rango de IPs disponibles para asignación |
| Cantidad de IPs del pool | Número total de direcciones IP disponibles |
| MAC del cliente | MAC usada para identificar y administrar las asignaciones |

---

## 🚀 Cómo Ejecutar el Script

bash
sudo python3 dhcp_spoofing.py


> ⚠️ *Debe ejecutarse con permisos root (sudo)*

---

## 📋 Funcionamiento del Script

*Paso 1:* El script comprueba que el usuario tenga permisos de administrador (root), ya que la captura y el envío de tramas de red requieren privilegios elevados.

*Paso 2:* Se solicita la *interfaz de red, la **IP del servidor falso, el **gateway falso, el **DNS falso, la **máscara de subred* y el *rango de IPs* a asignar.

*Paso 3:* El script genera un *pool de direcciones IP* disponibles y obtiene automáticamente la MAC de la interfaz del servidor falso.

*Paso 4:* El servidor DHCP falso comienza a *escuchar solicitudes DHCP* (Discover, Request y Release) de los clientes en la red.

*Paso 5:* Al recibir un *DHCP Discover, responde con un DHCP Offer. Al recibir un **DHCP Request*, confirma la asignación con un DHCP ACK, entregando el gateway y DNS del atacante.

*Paso 6:* Durante la ejecución, se muestran en pantalla las solicitudes recibidas, las IPs asignadas y las configuraciones de red entregadas a cada cliente.

---

## 📸 Capturas de Pantalla

### IP de Kali Linux proporcionada por el servidor DHCP legítimo
![IP Kali antes del ataque](https://res.cloudinary.com/drzcveg06/image/upload/v1780767964/DSPOOFING-CAP-1_fjtmuw.png)

![IP Kali antes del ataque 2](https://res.cloudinary.com/drzcveg06/image/upload/v1780767992/DSPOOFING-CAP-1-1_lnwetz.png)

### Ataque corriendo
![Ataque corriendo](https://res.cloudinary.com/drzcveg06/image/upload/v1780768031/DSPOOFING-CAP-2_d8d9qf.png)

### Kali Linux luego de correr el ataque
![Kali después del ataque](https://res.cloudinary.com/drzcveg06/image/upload/v1780768060/DSPOOFING-CAP-3_f3sngg.png)

---

## 🌐 Documentación de la Red

| Dispositivo | Interfaz | Dirección IP | Máscara de Red |
|-------------|----------|--------------|----------------|
| R-1 | E0/0 | 6.6.1.1 | 255.255.255.0 |
| SW-1 | ---- | ----- | ----- |
| Kali Linux (Atacante) | e1 | 6.6.1.13 | 255.255.255.0 |
| Windows 7 (Víctima) | e0 | 6.6.1.11 | 255.255.255.0 |
| Cloud | net | 192.168.206.135 | 255.255.255.0 |

---

## 🛡️ Contramedidas para Mitigar el Ataque

### 1. DHCP Snooping
Función configurada en el switch que clasifica los puertos en *trusted* y *untrusted*. Solo el puerto marcado como trusted puede enviar respuestas DHCP. Cualquier respuesta proveniente de un puerto untrusted es descartada automáticamente.

```
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1
SW1(config)# no ip dhcp snooping information option
SW1(config)# interface e0/0
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit
```

![DHCP Snooping resultado](https://res.cloudinary.com/drzcveg06/image/upload/v1780768089/DSPOOFING-M-1_hmdvdu.png)

### 2. IP Source Guard
Verifica que la dirección IP origen de cada paquete que entra por un puerto coincida con la información registrada en la tabla de DHCP Snooping. Si no coincide, el paquete es bloqueado antes de ser procesado.

```
SW1(config)# interface range e0/1 , e0/3
SW1(config-if-range)# ip verify source
SW1(config-if-range)# exit
```

![IP Source Guard resultado](https://res.cloudinary.com/drzcveg06/image/upload/v1780768116/DSPOOFING-M-2_ba1rdz.png)

---

> ⚠️ *Este script es únicamente con fines educativos y de investigación en entornos controlados.*  
> ⚠️ *BY: Eliannyy*
