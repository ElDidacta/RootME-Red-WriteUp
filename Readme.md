# RootME-Red-WriteUp 🛡️🌐

<img width="1892" height="367" alt="image" src="https://github.com/user-attachments/assets/feb90268-b624-46db-96a6-c84268be2c7e" />


Laboratorios prácticos y documentación técnica de **análisis forense de red**, **inspección profunda de paquetes (PCAP)** y **auditoría de protocolos** para la mitigación de riesgos e incidentes en infraestructuras corporativas distribuidas.

---

## 🎯 Objetivos del Repositorio

El propósito de este espacio es documentar metodologías de análisis ante incidentes de seguridad en entornos de red, proporcionando write-ups detallados y soluciones a escenarios reales o controlados (CTFs, laboratorios corporativos).

## 🛠️ Herramientas Utilizadas

Durante los análisis y auditorías se emplean las siguientes herramientas del ecosistema de ciberseguridad:
* **Wireshark / Tshark** - Inspección profunda y filtrado de paquetes.
* **Zeek (Bro) / Suricata** - Análisis de tráfico a gran escala y sistemas de detección de intrusos (IDS).
* **NetworkMiner** - Análisis forense de red pasivo y extracción de artefactos.
* **Scapy** - Manipulación y creación personalizada de paquetes de red.
* **hashcat** - Software de línea de comandos diseñada para la recuperación y el descifrado de contraseñas a partir de sus valores has.

## 📁 Estructura del Proyecto

```text
├── red rootme/
│   ├── Caso_01_Anomalia_HTTP/
│   │   ├── evidencias/          # Capturas de pantalla o extractos
│   │   ├── trafico_sospechoso.pcap
│   │   └── WRITEUP.md           # Resolución y análisis detallado
│   └── Caso_02_Exfiltracion_DNS/
└── Léame.md
```

## 📝 Metodología de Análisis General

Cada write-up incluido en este repositorio sigue un estándar riguroso:
1. **Identificación:** Revisión de alertas iniciales o anomalías reportadas.
2. **Aislamiento del Tráfico:** Aplicación de filtros de visualización (BPF/Wireshark) específicos para reducir el ruido.
3. **Línea de Tiempo:** Reconstrucción cronológica de los eventos y conexiones sospechosas.
4. **Impacto y Mitigación:** Determinación del alcance del incidente y recomendaciones técnicas para asegurar la infraestructura.

---
⚠️ *Descargo de responsabilidad: Todo el material expuesto en este repositorio tiene fines estrictamente educativos y de investigación.*

