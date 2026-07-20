# Phishing Case: Suplantación de Identidad de Indeed y Cress-Sport IT

**Autor:** Pablo Rodríguez Sánchez
**Fecha:** 20 de julio de 2026
**Clasificación:** Phishing / Ingeniería Social
**Estado:** Reportado a la plataforma afectada

---

## 📋 Resumen Ejecutivo

Este documento describe un intento de phishing altamente sofisticado que combina la suplantación de la plataforma de empleo Indeed con la identidad de una empresa real (Cress-Sport IT) para engañar a candidatos en busca de trabajo. La víctima había solicitado activamente una oferta publicada en Indeed, lo que añade una capa adicional de credibilidad al ataque.

El ataque utiliza tácticas de presión psicológica, redirecciones mediante código QR y dominios de vida efímera para evadir los sistemas de detección tradicionales.

El caso fue detectado a tiempo y no se produjo la instalación del software malicioso ni el compromiso del dispositivo de la víctima.

---

## 🎯 Vector de Ataque

| Elemento | Descripción |
|----------|-------------|
| **Canal de entrada** | Correo electrónico |
| **Remitente** | `Cress-Sport IT <no-reply@indeed.com>` |
| **Asunto** | Confirmación de preselección para oferta laboral |
| **URL del correo** | `pablosanchezs23fg_76e.vistatechno.com` |
| **URL intermedia (QR)** | `improve-platform.com` |
| **URL final** | `76e.vistatechno.com` |
| **IP del servidor** | `104.21.39.143` (Cloudflare, EE.UU.) |
| **Objetivo final** | Instalación de app de acceso remoto / robo de identidad |

---

## 🔍 Desarrollo del Ataque

### 1. Postulación inicial (legítima)

La víctima se postuló a una oferta de empleo publicada en la plataforma Indeed para la empresa **Cress-Sport IT**, con sede en Madrid. La oferta correspondía a un puesto de **Especialista en Seguridad TI** con un salario de **3.400 €/mes**.

### 2. Correo de contacto (fraudulento)

Días después de la postulación, la víctima recibió un correo electrónico que aparentaba ser la continuación legítima del proceso de selección:

> **De:** `Cress-Sport IT <no-reply@indeed.com>`
> **Para:** `[CORREO CENSURADO]`
> **Asunto:** Confirmación de preselección para oferta laboral
>
> *"Nos complace informarle que ha sido seleccionado(a) para nuestra lista de candidatos preseleccionados...*
>
> *Tenga en cuenta que los horarios de entrevista son muy limitados. Para garantizar y confirmar su espacio, le solicitamos amablemente que elija una fecha y hora dentro de las próximas 48 a 72 horas, antes de que el sistema de programación se cierre.*
>
> *Reserve su entrevista aquí:*  
> `pablosanchezs23fg_76e.vistatechno.com`*

### 3. Página intermedia con código QR

Al acceder al enlace, la víctima fue redirigida a `improve-platform.com`, que mostraba:

> **"Scan to continue!**  
> *To ensure the best functionality, this site requires a touch screen. Scan the code to open it on your phone."*

**Táctica de evasión:** El código QR traslada a la víctima del ordenador al teléfono, dificultando el análisis de la URL y aumentando el riesgo de instalación de apps maliciosas.

![Página QR](images/02-pagina-qr.png)

### 4. Página fraudulenta final

Al escanear el QR, la víctima accedía a `76e.vistatechno.com`, una página que:

- Utilizaba logos e imagen corporativa de Indeed
- Simulaba ser el portal oficial de reserva de entrevistas
- Solicitaba la descarga de una aplicación ("Indeed Interview")
- Pedía conectarse a una VPN e introducir un código de invitación

Este patrón es característico de estafas que buscan instalar software de acceso remoto (AnyDesk, TeamViewer o variantes).

---

## 🔬 Análisis Técnico

### 1. Análisis con urlscan.io (Página QR: improve-platform.com)

| Parámetro | Resultado |
|-----------|-----------|
| **URL analizada** | `https://improve-platform.com/` |
| **IP** | `104.21.39.143` (Cloudflare, EE.UU.) |
| **Ubicación** | Isla Ascensión |
| **Antigüedad del dominio** | **17 SEGUNDOS** |
| **Certificado SSL** | Emitido el mismo día, válido 3 meses |
| **Título de la página** | "Please wait" |
| **Contenido** | "Scan to continue!..." |
| **Veredicto** | Sin clasificación |

![urlscan.io analysis](images/03-urlscan-analysis.png)

### 2. Análisis con PowerDMARC (URL final: 76e.vistatechno.com)

| Parámetro | Resultado |
|-----------|-----------|
| **URL analizada** | `https://76e.vistatechno.com/` |
| **Índice de Confianza** | **100 / 100** (aparentemente seguro) |
| **OpenPhish** | No detectado |
| **Abuse.ch URLhaus** | No detectado |
| **Google Safe Browsing** | No detectado |
| **Antigüedad del dominio** |  No detectado como sospechoso |

> **Nota de PowerDMARC:** *"Un resultado «limpio» significa que no se ha detectado ninguna amenaza conocida en el momento de la comprobación. Aparecen constantemente nuevas páginas de phishing; en caso de duda, no las visites."*

![PowerDMARC analysis](images/04-powerdmarc-analysis.png)

### 3. Análisis comparativo de herramientas

| Herramienta | Resultado | Conclusión |
|-------------|-----------|------------|
| **urlscan.io** | Dominio de 17 segundos, sin clasificar | 🔴 Sospechoso |
| **PowerDMARC** | Índice 100/100, sin detecciones | 🟢 Aparentemente seguro |
| **Google Safe Browsing** | Sin clasificación | 🟡 Sin datos |

**Conclusión técnica:**

El dominio `76e.vistatechno.com` **no estaba catalogado como malicioso** en el momento del ataque. Esto significa que:

1. Los sistemas de seguridad tradicionales no lo bloqueaban
2. Los navegadores no mostraban advertencias
3. Los usuarios no tenían forma de saber que era peligroso

**Esta es la clave del ataque:** El dominio era tan nuevo que no había sido catalogado por ninguna base de datos de amenazas. Los estafadores aprovecharon esta ventana de oportunidad para atacar antes de que los sistemas de seguridad pudieran reaccionar.

---

##  Mapa completo de la infraestructura del ataque

Correo de phishing
De: Cress-Sport IT no-reply@indeed.com
↓
Enlace: pablosanchezs23fg_76e.vistatechno.com
↓
Página intermedia: improve-platform.com
urlscan.io: 17 segundos, IP Cloudflare (Isla Ascensión)
↓ "Scan to continue!"
Código QR (escaneado con el teléfono)
↓
Página fraudulenta: 76e.vistatechno.com
PowerDMARC: Índice 100/100 - NO detectado
↓
Solicita descarga de app maliciosa
↓
Solicita VPN + código de invitación
↓
COMPROMISO DEL DISPOSITIVO (si se completan los pasos)


---

## 🚨 Señales de Alerta Identificadas

### 🔴 1. Inconsistencia en el remitente (SEÑAL CRÍTICA)

| Elemento | Problema |
|----------|----------|
| **Nombre del remitente** | `Cress-Sport IT` |
| **Dominio del correo** | `@indeed.com` |

Un correo de una empresa **nunca** usa el dominio de una plataforma externa. Debería ser `@cress-sport.com`, **no `@indeed.com`**.

### 🔴 2. Tácticas de urgencia y presión

- *"los horarios de entrevista son muy limitados"*
- *"dentro de las próximas 48 a 72 horas"*
- *"antes de que el sistema de programación se cierre"*

### 🔴 3. Uso de código QR como técnica de evasión

El QR traslada a la víctima del ordenador al teléfono, donde:
- Es más difícil analizar la URL
- Es más fácil instalar apps maliciosas
- Se evaden sistemas de seguridad de escritorio

### 🔴 4. Dominios de vida efímera

| Dominio | Antigüedad | Estado |
|---------|-----------|--------|
| `improve-platform.com` | 17 segundos | 📸 URLscan |
| `76e.vistatechno.com` | No catalogado | 📸 PowerDMARC |

**Ambos dominios fueron creados minutos antes del ataque** para evitar su detección.

### 🔴 5. Proceso de entrevista atípico

| Proceso legítimo | Proceso fraudulento |
|------------------|---------------------|
| Zoom, Teams, Meet | App desconocida |
| Enlace directo | QR + redirecciones |
| Sin instalación | VPN + código de invitación |
| Sin acceso remoto | Solicitan control del dispositivo |

### 🔴 6. Beneficios inusuales en la oferta

"Uniforme proporcionado" para un puesto de **Especialista en Seguridad TI**: incompatible con el perfil. La oferta fue creada sin conocimiento del sector.

---

## 🤔 ¿Cómo es posible que los estafadores supieran que la víctima se había postulado?

### Hipótesis más probable: La oferta fue creada por los estafadores

La oferta publicada en Indeed fue creada por los atacantes para:
- Atraer candidatos reales
- Recibir currículums con datos personales
- Saber exactamente quiénes se habían postulado

**La postulación previa aumenta significativamente la credibilidad del correo.**

---

## 📸 Evidencias Recopiladas

| Evidencia | Descripción | Estado |
|-----------|-------------|--------|
| Correo electrónico | `Cress-Sport IT <no-reply@indeed.com>` | ✅ Disponible |
| Captura página QR | `improve-platform.com` | ✅ Disponible |
| Análisis urlscan.io | 17 segundos, IP Cloudflare | ✅ Disponible |
| Análisis PowerDMARC | Índice 100/100 | ✅ Disponible |
| Oferta de empleo | Publicada en Indeed | ✅ Disponible |

---

## 🛡️ Medidas de Mitigación

### Para la víctima (aplicadas)

- ✅ No se descargó ni instaló la aplicación
- ✅ No se introdujo ningún código
- ✅ No se compartió acceso al dispositivo
- ✅ Se reportó a Indeed
- ✅ Se analizó la URL con urlscan.io y PowerDMARC

### Recomendaciones generales

1. **Verificar siempre el remitente** - El dominio debe coincidir con la empresa
2. **Desconfiar de las urgencias** - Las empresas serias no presionan así
3. **No descargar apps externas** - Las entrevistas usan Zoom, Teams o Meet
4. **Nunca compartir acceso remoto** - Ningún proceso legítimo lo solicita
5. **No escanear QR de fuentes desconocidas**
6. **Analizar enlaces con urlscan.io o VirusTotal**
7. **Verificar la oferta en la plataforma oficial**
8. **Contactar a la empresa por canales oficiales**

---

## 📊 Lecciones Aprendidas

### Lo que hizo bien la víctima

- Detectó la inconsistencia en el remitente
- Reconoció tácticas de presión
- Consultó antes de actuar
- Documentó todo el proceso
- No completó los pasos solicitados

### Lo que los atacantes hicieron "bien" (desde su perspectiva)

- Usaron dominio legítimo (`@indeed.com`)
- Suplantaron empresa real
- Crearon urgencia
- Usaron logos creíbles
- Diseñaron flujo lógico
- **Explotaron postulación real**
- **Usaron QR como evasión**
- **Dominios de 17 segundos**
- **Dominios NO catalogados como maliciosos**

---

##  Conclusiones

Este caso de phishing destaca por su **sofisticación técnica y de ingeniería social** al combinar:

1. Una **postulación real** de la víctima (expectativa)
2. Una **plataforma legítima** (Indeed) como fachada
3. Una **empresa real** (Cress-Sport IT) como señuelo
4. Un **proceso de selección simulado** para dar credibilidad
5. **Tácticas de presión** para evitar el pensamiento crítico
6. **Código QR** como técnica de evasión
7. **Dominios de vida efímera** (17 segundos)
8. **Dominios NO catalogados** en bases de datos de amenazas
9. **Servicios de proxy** (Cloudflare) para ocultamiento

### La paradoja de la seguridad

> **El dominio tenía un Índice de Confianza de 100/100 en PowerDMARC.**  
> **Y sin embargo era fraudulento.**

Esto demuestra que:
- Las herramientas de seguridad no son infalibles
- Un dominio "limpio" no significa seguro
- La velocidad del ataque supera a la capacidad de detección
- **El factor humano sigue siendo la mejor defensa**

**La diferencia entre caer o no fue prestar atención al remitente.**

---

## 🔗 Referencias

- [Indeed Centro de Seguridad](https://www.indeed.com/security)
- [Incibe - Phishing](https://www.incibe.es/protege-tu-empresa/amenazas/phishing)
- [urlscan.io](https://urlscan.io/)
- [PowerDMARC](https://powerdmarc.com/)
- [CRESS Sport (empresa real)](https://www.cress-sport.com/)

---

## Indicadores de Compromiso (IOCs)

Disponibles en [iocs.txt](iocs.txt)

---

*Este writeup ha sido elaborado con fines educativos y de concienciación sobre ciberseguridad.*

**#Phishing #Cybersecurity #Indeed #CressSportIT #SecurityAwareness #JobScam #OSINT #SocialEngineering #Cloudflare #urlscan #PowerDMARC**
