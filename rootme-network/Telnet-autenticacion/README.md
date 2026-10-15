# 📊 Auditoría de Seguridad en Red: TELNET - Autenticación (Root-Me)
<img width="702" height="702" alt="image" src="https://github.com/user-attachments/assets/63c7e720-220d-4b35-a156-77b6f41960ec" />

* **Plataforma:** Root-Me.org
* **Categoría:** Network / Redes de Datos
* **Dificultad:** Muy Fácil (5 Puntos)
* **Herramientas utilizadas:** Wireshark v4.x

---

## 🎯 Objetivo de la Auditoría
El propósito de este laboratorio es analizar una captura de tráfico de red empaquetada en un archivo `.pcap` para interceptar y evaluar las credenciales de administración remota de un equipo de red, documentando las vulnerabilidades críticas y estructurales que presenta el protocolo Telnet al transmitir información en texto claro por el cable.

---

## 🔍 Proceso de Resolución y Metodología (Paso a Paso)

### Paso 1: Validación Inicial del Desafío
Se inició el laboratorio interactivo en la plataforma Root-Me para descargar la telemetría base del tráfico de red empaquetado en el archivo `ch2.pcap`.


<img width="1317" height="977" alt="image" src="https://github.com/user-attachments/assets/6b2e5336-0bff-4cc2-9b6e-d90857c3dd5b" />

### Paso 2: Análisis del Tráfico en Bruto
Al abrir la captura en Wireshark, se detectó una ráfaga multiplexada de tramas del protocolo TCP interactivo en los puertos de control base, evidenciando un intercambio constante de bytes entre las direcciones de red locales `192.168.0.2` y `192.168.0.1`.

<img width="1306" height="957" alt="image" src="https://github.com/user-attachments/assets/ae13182a-770b-4d2e-839b-2dbf4ca4f75f" />


### Paso 3: Aislamiento del Protocolo Inseguro
Para limpiar el ruido de la captura y aislar de forma quirúrgica el momento exacto en que el operador inicia la solicitud de acceso al sistema, se aplicó un filtro avanzado de visualización en la barra superior apuntando a la cadena de autenticación:
```query
telnet contains "login"
```
Este comando redujo el espectro de visualización a únicamente tres paquetes críticos de control (Paquetes N° 28, N° 70 y N° 116), confirmando el intento de negociación sobre una consola OpenBSD.

<img width="1316" height="927" alt="image" src="https://github.com/user-attachments/assets/b2f72a88-9d04-40db-9779-7cb06940e916" />


### Paso 4: Inspección Profunda (Deep Packet Inspection) e Intercepción
Dado que Telnet opera bajo la especificación **RFC 854** transmitiendo los caracteres de forma interactiva (secuencias de `1 byte data`), la información viaja fragmentada en el cable de red. Para reconstruir la sesión de forma legible y limpia como un flujo de texto único, se seleccionó el paquete N° 28 y se ejecutó la acción:
```text
Click Derecho ➔ Seguir ➔ Flujo TCP (Follow TCP Stream)
```
Al abrirse la ventana emergente, se interceptaron en texto claro las credenciales administrativas de acceso remoto a la terminal virtual (VTY):
* **Identificador de Acceso (Usuario):** `login: fake`
* **Credencial Crítica (Contraseña):** `Password: [REDACTED_FOR_SECURITY]`

*Análisis post-autenticación:* Se observó que tras vulnerar el acceso, el operador ejecutó comandos de reconocimiento local (`ls -a`) y verificó conectividad perimetral externa mediante solicitudes ICMP (`/sbin/ping ://yahoo.com`), demostrando que un sniffer pasivo en la red adquiere visibilidad absoluta de las acciones tácticas del administrador.
<img width="1337" height="972" alt="image" src="https://github.com/user-attachments/assets/cbeea1b5-d181-4493-b107-421ed181b559" />


<img width="1210" height="867" alt="image" src="https://github.com/user-attachments/assets/2a6bfa62-c845-43d7-88cf-383ab8fac4f4" />


<img width="667" height="757" alt="image" src="https://github.com/user-attachments/assets/2fd780b3-7973-4a22-a831-787c20a69690" />

---

## 🛡️ Solución de Ingeniería Aplicada y Remediación (Caso Corporativo)

Para mitigar esta falta de seguridad de raíz y blindar la administración remota de los enlaces y routers en una infraestructura distributed (como plantas de acopio, sucursales rurales o terminales administrativas de la cooperativa), se aplica un modelo de remediación en la capa de red:

### 1. Desmantelamiento del Demonio Inseguro
Se dicta la prohibición total del protocolo Telnet (puerto 23) para cualquier tarea de gestión de infraestructura corporativa, procediendo a su apagado absoluto en los sistemas operativos y equipos perimetrales.

### 2. Endurecimiento Perimetral en RouterOS para evitar que sea un acceso y realicen pivoting y comprometan toda la red. 
Se aplican políticas restrictivas a nivel de hardware en los routers **MikroTik** de la empresa para deshabilitar el servicio nativo inseguro y forzar el uso exclusivo de consolas cifradas mediante criptografía asimétrica robusta (**SSH - Puerto 22**):

```cmd
# 1. Desactivar de forma mandatoria el servicio de servidor Telnet en RouterOS
/ip service disable telnet

# 2. Habilitar y forzar el uso exclusivo del canal seguro SSH para los ingenieros
/ip service enable ssh
/ip service set ssh port=22 comment="Administracion cifrada de extremo a extremo"

# 3. Aplicar regla de firewall perimetral para dropear intentos de escaneo sobre el puerto 23
/ip firewall filter add chain=forward protocol=tcp dst-port=23 action=drop comment="Bloquear e identificar trafico Telnet en la red"
```

### 3. Centralización de Telemetría (Wazuh SIEM + Zabbix)
* **Monitoreo con Zabbix:** Configurar alertas (*triggers*) basadas en telemetría SNMP sobre los MikroTik para detectar intentos de conexión o tráfico inusual en el puerto 23, alertando de inmediato sobre posibles escaneos de red internos.
* **Correlación con Wazuh:** El agente de Wazuh centraliza los logs de autenticación SSH de los servidores principales de Rosario, auditando inicios de sesión exitosos fuera del horario comercial o bloqueando IPs de forma automatizada tras detectar intentos fallidos recurrentes.
