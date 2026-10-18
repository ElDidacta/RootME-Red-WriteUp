# Root-Me - Kerberos Authentication Write-Up

## Descripción del Reto
Este laboratorio de **Root-Me** se enfoca en el análisis de un flujo de autenticación de **Kerberos** capturado en tráfico de red (`kerberos_auth.pcapng`), permitiendo comprender el intercambio de mensajes AS-REQ/AS-REP y la ejecución de ataques offline contra la pre-autenticación de Kerberos (*AS-REP Roasting* / descifrado de `PA-ENC-TIMESTAMP`).

---

<img width="835" height="837" alt="image" src="https://github.com/user-attachments/assets/3a306ab4-c483-4523-b712-2c520afde2b9" />

---

## 1. Análisis de Tráfico (Wireshark)

1. **Inspección inicial**: Al abrir el archivo `.pcapng` en Wireshark, filtramos o visualizamos los paquetes con protocolo `KRB5` para identificar las peticiones de autenticación inicial:
   - `AS-REQ` y `AS-REP` (Intercambio inicial entre el cliente y el KDC).

 <img width="1312" height="597" alt="image" src="https://github.com/user-attachments/assets/b2759f03-8ea4-4a40-9cf8-8d86193c7a34" />



2. **Extracción de datos del paquete AS-REQ (`Frame 60`)**:
   - Analizamos la estructura donde se envía el `PA-ENC-TIMESTAMP` cifrado.
   - **Usuario objetivo**: `william.dupond`[cite: 3]
   - **Dominio**: `CATCORP.LOCAL`[cite: 3]
   - **Tipo de cifrado**: `ETYPE-AES256-CTS-HMAC-SHA1-96` (Etype 18)[cite: 2, 3].

<img width="1877" height="1032" alt="image" src="https://github.com/user-attachments/assets/135ca5d5-3969-4202-99fa-c80b0d90f104" />

---

<img width="1297" height="696" alt="image" src="https://github.com/user-attachments/assets/f1c6c99c-b07e-4c08-8607-15d8fdfdcf31" />


---

## 2. Explotación y Descifrado (Hashcat)

Una vez extraído el hash del tráfico capturado, procedimos a realizar un ataque de fuerza bruta por diccionario utilizando **Hashcat** junto con el diccionario `rockyou.txt` para recuperar la clave en texto plano.

1. **Comando ejecutado**:
   ```bash
   hashcat -m 19900 rootme.txt -a 0 /home/kali/rockyou.txt

<img width="955" height="970" alt="image" src="https://github.com/user-attachments/assets/4fd0dd60-d12d-4d07-8c18-3613bf7fec0b" />

---

<img width="827" height="802" alt="Captura de pantalla 2026-10-18 204736" src="https://github.com/user-attachments/assets/ab1e2dee-7094-4f87-af35-0505263b7317" />

