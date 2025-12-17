# SSL/TLS Man-in-the-Middle (MITM)
Nombre del ataque

SSL/TLS Man-in-the-Middle (MITM)

Descripción

Este ataque consiste en interceptar comunicaciones cifradas entre un cliente y un servidor web. El atacante se posiciona como intermediario dentro de la red local mediante técnicas de ARP Spoofing y redirección de tráfico, utilizando un proxy transparente para descifrar y analizar el contenido.


Protocolos afectados

ARP (Address Resolution Protocol)
Se manipula para redirigir el tráfico de la víctima hacia el atacante.

IP / TCP
Se alteran las rutas de comunicación mediante reglas de iptables.

SSL/TLS (HTTPS)
Se rompe la confianza del canal cifrado instalando un certificado falso en la víctima, permitiendo el descifrado del tráfico.


Impacto.

Captura de credenciales en texto plano.
Pérdida de confidencialidad de las comunicaciones seguras.
Posibilidad de manipulación del contenido transmitido entre cliente y servidor.


Condiciones necesarias.

Acceso a la misma red local que la víctima.
Instalación de un certificado falso en el sistema víctima para evitar alertas de seguridad.


Mitigación

Uso de HSTS para forzar conexiones HTTPS legítimas.
Monitorización de ARP para detectar envenenamiento.
Uso de certificados válidos y verificación estricta en navegadores.
Segmentación de red y detección de proxies no autorizados.


Relación protocolo–ataque

| Protocolo        | Vulneración técnica                                                             | Rol en el ataque                                                                 |
|------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| ARP              | ARP Spoofing para engañar a la víctima y al router                              | Permite posicionar al atacante como intermediario en la red local (MITM físico) |
| HTTPS (SSL/TLS)  | Intercepción mediante proxy transparente y certificado falso                    | Permite descifrar comunicaciones seguras (credenciales, sesiones)               |
| IP / TCP         | Redirección de tráfico con reglas iptables                                      | Facilita la interceptación del tráfico antes de llegar al destino               |


Explicación paso a paso

ARP Spoofing
Bettercap envía respuestas ARP falsas para que la víctima crea que Kali es el router y viceversa. Esto explota la falta de autenticación del protocolo ARP.

Redirección de tráfico
Kali utiliza iptables para redirigir todo el tráfico HTTP (puerto 80) y HTTPS (puerto 443) al puerto 8080, donde mitmproxy lo intercepta.

Intercepción SSL/TLS
Al instalar el certificado falso en la víctima, el navegador confía en Kali como si fuera una autoridad certificadora legítima, permitiendo descifrar el tráfico HTTPS.


Topología y direcciones IP

Kali (Atacante): 192.168.0.25
Dispositivo que intercepta y analiza el tráfico.

<img width="811" height="276" alt="IP Kali" src="https://github.com/user-attachments/assets/398ce78f-900a-4659-bdb2-4fd4d89bd074" />


Ubuntu (Víctima): 192.168.0.26
Equipo objetivo del cual se capturan las credenciales.

<img width="841" height="251" alt="IP linux victima" src="https://github.com/user-attachments/assets/eb1e8fca-0ceb-42ab-b392-3b17040ef8de" />


Router / Gateway: 192.168.0.1
Puerta de enlace hacia Internet.

Interfaz Kali: eth0
Interfaz de red utilizada para el ataque.


FASE 0: Preparación y limpieza del entorno

A. Limpieza de red (Kali – T1)

Restaurar IPTables -> sudo iptables -t nat -F
Elimina reglas previas de redirección para evitar conflictos.

Deshabilitar IP Forwarding -> echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
Garantiza un estado inicial neutro del sistema.

B. Actualización e instalación (Kali – T1)

Actualizar el sistema -> sudo apt update && sudo apt upgrade -y
Instalar herramientas necesarias-> sudo apt install bettercap mitmproxy -y

<img width="1099" height="565" alt="Fase 0 kali" src="https://github.com/user-attachments/assets/f8a60a4f-800d-471c-a5de-abdf3c5a8761" />


FASE 1: Configuración inicial del ataque (Kali – T1)
A. Preparación del certificado falso (CA)

1-Generar el certificado raíz de mitmproxy

Con el comando -> mitmproxy
Una vez cargada la herramienta-> (Salir con q y confirmar con y. Se crea ~/.mitmproxy/)

2-Copiar el certificado para fácil acceso

cp ~/.mitmproxy/mitmproxy-ca-cert.pem ~/mitmproxy-ca.pem

B. Configuración del núcleo de red

Habilitar IP Forwarding -> echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
Redirección de tráfico HTTP-> sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
Redirección de tráfico HTTPS-> sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8080

<img width="1115" height="397" alt="image" src="https://github.com/user-attachments/assets/0f7c3d51-0e34-4ab1-8a73-2ce7357c38e8" />

FASE 2: Configuración de la víctima (Linux)

A. Transferencia del certificado-> scp ~/mitmproxy-ca.pem usuario@192.168.0.X:/home/usuario/

<img width="767" height="355" alt="Fase 1 kali" src="https://github.com/user-attachments/assets/b71c0a04-8dc5-463e-ac0a-690cb288cf30" />

B. Instalación del certificado de confianza

Copiar certificado al almacén del sistema-> sudo cp mitmproxy-ca.pem /usr/local/share/ca-certificates/mitmproxy-ca.crt

Actualizar certificados de confianza-> sudo update-ca-certificates

<img width="900" height="151" alt="Fase 2 linux parte 1" src="https://github.com/user-attachments/assets/abd652d9-7429-46dd-b4c3-2e3831d97aba" />

Configuración del navegador (Firefox)
Importar el certificado desde:
Ajustes → Certificados -> Importar Autoridad certificadora.

<img width="1156" height="524" alt="Fase 2 linux parte 2 " src="https://github.com/user-attachments/assets/1ae19641-dba4-4dcb-bdc2-d3515f1db8e1" />

Añadimos el certificado creado.

<img width="766" height="336" alt="Fase 2 linux parte 3" src="https://github.com/user-attachments/assets/03f8e2e1-bfb5-4270-99d6-1dbf3e0861a7" />

Aceptamos en la casilla-> "Trust this CA to indentity websites" -> Confiar en esta autoridad certificadora.

<img width="726" height="407" alt="Fase 2 linux parte 4" src="https://github.com/user-attachments/assets/a4ba5821-5701-4c51-8c29-8ae54750a6e9" />


FASE 3: Ejecución del ataque MITM (Kali)

A. Iniciar el proxy-> sudo mitmproxy --mode transparent
B. Iniciar el ARP Spoofing 

Lanzar Bettercap-> sudo bettercap -iface eth0

Definir objetivos-> set arp.targets 192.168.0.26,192.168.0.1

Activar ARP Spoofing-> arp.spoof on













