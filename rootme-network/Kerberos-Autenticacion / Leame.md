# Root-Me - Kerberos Authentication Write-Up

## Descripción del Reto
Este laboratorio de **Root-Me** se enfoca en el análisis de un flujo de autenticación de **Kerberos** capturado en tráfico de red (`kerberos_auth.pcapng`), permitiendo comprender el intercambio de mensajes AS-REQ/AS-REP y la ejecución de ataques offline contra la pre-autenticación de Kerberos (*AS-REP Roasting* / descifrado de `PA-ENC-TIMESTAMP`).

---

## 1. Análisis de Tráfico (Wireshark)

1. **Inspección inicial**: Al abrir el archivo `.pcapng` en Wireshark, filtramos o visualizamos los paquetes con protocolo `KRB5` para identificar las peticiones de autenticación inicial:
   - `AS-REQ` y `AS-REP` (Intercambio inicial entre el cliente y el KDC).

   ![Flujo de paquetes Kerberos en Wireshark](ruta_a_tu_imagen_wireshark_1.png)

2. **Extracción de datos del paquete AS-REQ (`Frame 60`)**:
   - Analizamos la estructura donde se envía el `PA-ENC-TIMESTAMP` cifrado.
   - **Usuario objetivo**: `william.dupond`[cite: 3]
   - **Dominio**: `CATCORP.LOCAL`[cite: 3]
   - **Tipo de cifrado**: `ETYPE-AES256-CTS-HMAC-SHA1-96` (Etype 18)[cite: 2, 3].

   ![Detalle del paquete AS-REQ y estructura cname/sname](ruta_a_tu_imagen_wireshark_2.png)

---

## 2. Explotación y Descifrado (Hashcat)

Una vez extraído el hash del tráfico capturado, procedimos a realizar un ataque de fuerza bruta por diccionario utilizando **Hashcat** junto con el diccionario `rockyou.txt` para recuperar la clave en texto plano.

1. **Comando ejecutado**:
   ```bash
   hashcat -m 19900 rootme.txt -a 0 /home/kali/rockyou.txt
