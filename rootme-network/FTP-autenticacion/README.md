# 📊 Auditoría de Seguridad en Red: FTP - Autenticación (Root-Me)
<img width="1647" height="767" alt="Captura de pantalla 2026-10-15 005641" src="https://github.com/user-attachments/assets/670f23d1-24c5-4360-9812-0e073b69df06" />

* **Plataforma:** Root-Me.org
* **Categoría:** Network / Redes de Datos
* **Dificultad:** Muy Fácil (5 Puntos)
* **Herramientas utilizadas:** Wireshark v4.x

---

## 🎯 Objetivo de la Auditoría
El propósito de este laboratorio es analizar una captura de tráfico de red empaquetada en un archivo `.pcap` para interceptar credenciales de acceso a un servidor corporativo distribuidor de archivos, evaluando las debilidades estructurales asociadas al uso de protocolos heredados que transmiten información en texto claro por el cable.

---

## 🔍 Proceso de Resolución y Metodología (Paso a Paso)

### Paso 1: Relevamiento del Entorno Inicial
Al abrir la captura original `ch1.pcap` en Wireshark, se observa una gran cantidad de ruido y paquetes multiplexados en la red. Domina el tráfico TCP base, el establecimiento de conexiones (Handshakes de 3 vías) y las respuestas de control de flujo, lo que dificulta la lectura directa de las acciones del usuario.
<img width="1917" height="1077" alt="Captura de pantalla 2026-10-15 005155" src="https://github.com/user-attachments/assets/61839685-c26a-43aa-8920-4d3dee9f3269" />



### Paso 2: Aislamiento del Protocolo Inseguro
Para limpiar el ruido de la captura, se aplicó un filtro de visualización en la barra superior apuntando específicamente al protocolo de capa de aplicación FTP (File Transfer Protocol):
```query
ftp
```
Al ejecutar el filtro, Wireshark redujo los 105 paquetes totales a únicamente **19 paquetes del protocolo FTP**, revelando la sesión completa de comandos entre la IP cliente (`10.20.144.151`) y la IP del servidor de archivos (`10.20.144.150`).

<img width="1917" height="1077" alt="Captura de pantalla 2026-10-15 005314" src="https://github.com/user-attachments/assets/cd345ba8-0a2f-4262-845a-62be36bcbcd7" />


### Paso 3: Análisis de Paquetes e Intercepción de Credenciales
Debido a que el protocolo FTP tradicional bajo la especificación **RFC 959** no implementa ninguna capa de cifrado criptográfico, los comandos de autenticación viajan expuestos directamente por el cable de red en texto claro. 

Inspeccionando la columna **Length** e **Info**, se ejecutó un análisis profundo sobre dos paquetes críticos de la negociación:
1. **Paquete N° 8 (Comando USER):** El cliente envía la petición de usuario revelando la identidad de la cuenta: `USER cdts3500`.
2. **Paquete N° 11 (Comando PASS):** Inmediatamente después, el cliente envía la solicitud de contraseña exponiendo la clave de acceso en texto plano por la red: `PASS cdts3500`.

*Nota de análisis técnico:* Se constata que el entorno sufría además de una debilidad por factor humano, al reutilizar el mismo identificador de usuario como contraseña del sistema (`cdts3500`).

<img width="1917" height="1077" alt="CAPTURA MARCADA" src="https://github.com/user-attachments/assets/87e782cf-7fa0-4452-9d6a-0fc3a8fefe2f" />


---

## 🛡️ Solución de Ingeniería Aplicada y Remediación (Caso Corporativo)

Para mitigar esta vulnerabilidad de raíz y evitar fugas de información sensible en una infraestructura distributed (como plantas de acopio, sucursales y terminales asociadas al entorno **SAP**), se implementa un modelo de defensa en tres capas operativas:

### 1. Reemplazo del Protocolo Inseguro (Cifrado de Extremo a Extremo)
Se dicta el desmantelamiento total del protocolo FTP clásico (puerto 21) en entornos de producción, reemplazándolo de forma obligatoria por:
* **SFTP (SSH File Transfer Protocol - Puerto 22):** Corre sobre una sesión SSH utilizando algoritmos criptográficos robustos como **AES**. Cifra el canal de autenticación y los datos en tránsito, volviendo el tráfico inmune a técnicas de Sniffing. Es la solución nativa para automatizar el intercambio de reportes hacia el ERP corporativo.

### 2. Endurecimiento de Seguridad Perimetral (Firewall en MikroTik RouterOS)
Se aplican políticas restrictivas a nivel de hardware mediante el perfil de **MTCNA / MTCSE** en los routers de acceso principal para bloquear y denegar cualquier intento de tráfico no cifrado:

```cmd
# 1. Bloquear de forma mandatoria todo tráfico FTP saliente (Capa de Aplicación)
/ip firewall filter add chain=forward protocol=tcp dst-port=21 action=drop comment="Bloquear trafico FTP inseguro - Migrar a SFTP"

# 2. Bloquear puertos de datos dinámicos asociados al FTP clásico
/ip firewall filter add chain=forward protocol=tcp dst-port=20 action=drop comment="Bloquear FTP-Data"

# 3. Permitir exclusivamente canales seguros de transferencia y administración por SSH/SFTP
/ip firewall filter add chain=forward protocol=tcp dst-port=22 action=accept comment="Permitir trafico seguro SFTP/SSH"
```

### 3. Monitoreo Centralizado y Alertas Tempranas (Wazuh SIEM + Zabbix)
* **Detección con Wazuh:** El agente de Wazuh instalado en el servidor de archivos audita los registros de eventos del sistema operativo. Si se detectan intentos de escaneo de puertos (reconocimiento sobre el puerto 21) o ataques de fuerza bruta, el SIEM genera una alerta en tiempo real en la central de Rosario.
* **Métricas con Zabbix:** Monitoreo mediante telemetría SNMP en los routers MikroTik para identificar picos anómalos de tráfico en puertos de administración, garantizando visibilidad total sobre la disponibilidad de los enlaces corporativos.
