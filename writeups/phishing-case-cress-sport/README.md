# Phishing Case: Suplantación de Identidad de Indeed y Cress-Sport IT

**Autor:** Pablo Rodríguez Sánchez
**Fecha:** 20 de julio de 2026
**Clasificación:** Phishing / Ingeniería Social / Abuso de Plataforma
**Estado:** Reportado a la plataforma afectada

---

##  Resumen Ejecutivo

Este documento describe un intento de phishing altamente sofisticado que explota la **propia plataforma de Indeed** para enviar mensajes fraudulentos a candidatos que se han postulado a ofertas de empleo reales. El ataque combina:

- Abuso del sistema de mensajería de Indeed
- Suplantación de una empresa real (Cress-Sport IT)
- Tácticas de ingeniería social (urgencia, preselección)
- Técnicas de evasión (códigos QR, dominios efímeros)
- Uso de infraestructura Cloudflare para ocultamiento

El caso fue detectado a tiempo y no se produjo la instalación del software malicioso ni el compromiso del dispositivo de la víctima.

**Este es un caso especialmente peligroso porque el mensaje se envió a través de los canales legítimos de Indeed, lo que dificulta su detección.**

---

##  Vector de Ataque

| Elemento | Descripción |
|----------|-------------|
| **Canal de entrada** | Sistema de mensajería de Indeed (legítimo) |
| **Remitente visible** | `Cress-Sport IT <no-reply@indeed.com>` |
| **Nombre del reclutador** | Theo Pierre Justin |
| **Asunto** | Mensaje nuevo de Theo Pierre Justin - Especialista en Seguridad TI |
| **Dominio de envío** | `bounces.indeed.com` (legítimo) |
| **Dominio del mensaje** | `messages.indeed.com` (legítimo) |
| **URL maliciosa en el mensaje** | `pablosanchez23fg.76evistatechno.com` |
| **URL intermedia (QR)** | `improve-platform.com` |
| **URL final** | `76e.vistatechno.com` |
| **IP del servidor** | `104.21.39.143` (Cloudflare, EE.UU.) |
| **Objetivo final** | Instalación de app de acceso remoto / robo de identidad |

---

##  Desarrollo del Ataque

### 1. Postulación inicial (legítima)

La víctima se postuló a una oferta de empleo publicada en la plataforma Indeed para la empresa **Cress-Sport IT**, con sede en Madrid. La oferta correspondía a un puesto de **Especialista en Seguridad TI** con un salario de **3.400 €/mes**.

### 2. El mensaje fraudulento a través de Indeed

Días después de la postulación, la víctima recibió un mensaje a través del **sistema de mensajería interno de Indeed**. El mensaje aparecía en la bandeja de entrada de Indeed como cualquier comunicación legítima de un empleador.

**Metadatos del mensaje:**

| Campo | Valor |
|-------|-------|
| **Remitente** | `Cress-Sport IT <no-reply@indeed.com>` |
| **Nombre del reclutador** | Theo Pierre Justin |
| **Asunto** | Mensaje nuevo de Theo Pierre Justin - Especialista en Seguridad TI |
| **Enviado por** | `bounces.indeed.com` (legítimo) |
| **Firmado por** | `indeed.com` (legítimo) |

**Contenido del mensaje:**

> *"Estimado(a) Nos complace informar que ha sido seleccionado(a) para nuestra lista de candidatos preseleccionados, y nos entusiasma avanzar con usted a la siguiente etapa de nuestro proceso de selección. Tenga en cuenta que los horarios de entrevista son muy limitados. Para garantizar y confirmar su espacio, le solicitamos amablemente que elija una fecha y hora dentro de las próximas 48 a 72 horas, antes de que el sistema de programación se cierre. Reserve su entrevista aquí: pablosanchez23fg.76evistatechno.com"*

> *"El enlace para la reunión se generará automáticamente una vez completada la reserva. Le sugerimos realizar este proceso a la brevedad. Quedamos a la espera de su confirmación y le deseamos el mayor de los éxitos."*

> *"Atentamente,"*

### 3. Análisis del enlace de Indeed

El enlace del mensaje apuntaba a una URL legítima de Indeed:
https://messages.indeed.com/conversaciones/[ID_LARGO]

**Análisis técnico del dominio de Indeed:**

| Parámetro | Resultado |
|-----------|-----------|
| **URL** | `https://messages.indeed.com/` |
| **IP** | `162.159.130.67` (Cloudflare, EE.UU.) |
| **Ubicación** | Isla Ascensión |
| **Antigüedad del dominio** | **7 años** (creado en 1998) |
| **Certificado SSL** | Emitido por WE1, válido por 3 meses |
| **Título de la página** | "Un momento..." |
| **Estado HTTP** | 403 (acceso denegado) |
| **Veredicto urlscan.io** | Sin clasificación |

> **Observación:** El enlace de Indeed fue escaneado en urlscan.io **el 20 de julio de 2024** (un año antes del ataque), lo que indica que los estafadores podrían haber estado probando su infraestructura con mucha antelación.

### 4. La URL maliciosa en el mensaje

El mensaje contenía el siguiente enlace fraudulento:
pablosanchez23fg.76evistatechno.com

**Observaciones sobre esta URL:**
- Utiliza un subdominio personalizado (`pablosanchez23fg`) para simular ser un enlace de reserva de entrevista
- El dominio base `76evistatechno.com` (variante de `vistatechno.com`) fue registrado para la estafa
- La URL no incluye el protocolo `https://`, lo que es inusual en comunicaciones legítimas

### 5. Redirección a la página QR

Al hacer clic en el enlace, la víctima era redirigida a una página intermedia:
improve-platform.com


Esta página mostraba:

> **"Scan to continue!**  
> *To ensure the best functionality, this site requires a touch screen. Scan the code to open it on your phone."*

**Análisis técnico:**

| Parámetro | Resultado |
|-----------|-----------|
| **URL** | `https://improve-platform.com/` |
| **IP** | `104.21.39.143` (Cloudflare) |
| **Ubicación** | Isla Ascensión |
| **Antigüedad del dominio** | **17 SEGUNDOS** |
| **Certificado SSL** | Emitido el mismo día |
| **Título** | "Please wait" |

### 6. Página fraudulenta final

Al escanear el QR, la víctima accedía a:
76e.vistatechno.com


Esta página:
- Utilizaba logos e imagen corporativa de Indeed
- Simulaba ser el portal oficial de reserva de entrevistas
- Solicitaba la descarga de una aplicación ("Indeed Interview")
- Pedía conectarse a una VPN e introducir un código de invitación

**Análisis con PowerDMARC:**

| Parámetro | Resultado |
|-----------|-----------|
| **Índice de Confianza** | **100 / 100** (NO detectado) |
| **OpenPhish** | No detectado |
| **Abuse.ch URLhaus** | No detectado |
| **Google Safe Browsing** | No detectado |

---

##  Análisis de Infraestructura

### Comparativa de dominios

| Dominio | Antigüedad | Propósito | Estado |
|---------|-----------|-----------|--------|
| `messages.indeed.com` | 7 años | Plataforma legítima de Indeed |  Legítimo |
| `pablosanchez23fg.76evistatechno.com` | Desconocida | Enlace del mensaje |  Fraudulento |
| `improve-platform.com` | 17 segundos | Página con QR |  Fraudulento |
| `76e.vistatechno.com` | Desconocida | Página final |  Fraudulento |

### Mapa completo del ataque
Oferta de empleo publicada en Indeed (Cress-Sport IT)
↓

Víctima se postula (acto legítimo)
↓

Los estafadores (supuestamente "Theo Pierre Justin")
envían mensaje a través del sistema de Indeed
De: Cress-Sport IT no-reply@indeed.com
Vía: bounces.indeed.com (legítimo)
↓

La víctima ve el mensaje en su bandeja de Indeed
(parece legítimo porque está en la plataforma)
↓

El mensaje contiene un enlace a:
pablosanchez23fg.76evistatechno.com
↓

Redirección a improve-platform.com
17 segundos de antigüedad
↓

Página QR: "Scan to continue!"
↓

Escaneo con teléfono → 76e.vistatechno.com
PowerDMARC: 100/100 (NO detectado)
↓

Página con logos de Indeed
Solicita descargar app + VPN + código
↓

COMPROMISO DEL DISPOSITIVO (si se completa)


---

##  Señales de Alerta Identificadas

###  1. El mensaje llega a través de Indeed (ABUSO DE PLATAFORMA)

El mensaje fue enviado a través del **sistema legítimo de mensajería de Indeed**. Esto es especialmente peligroso porque:

- La víctima confía en los mensajes de Indeed
- El mensaje aparece en la bandeja de entrada de Indeed
- Los metadatos del correo (`bounces.indeed.com`, `firmado por: indeed.com`) son legítimos

> **Conclusión:** Los estafadores **abusaron del sistema de Indeed** para enviar mensajes fraudulentos. Indeed permitió que un estafador enviara un mensaje a través de su plataforma suplantando a una empresa.

###  2. El nombre del reclutador "Theo Pierre Justin"

El nombre del supuesto reclutador es **"Theo Pierre Justin"**. No hay información pública que vincule este nombre con Cress-Sport IT.

###  3. Tácticas de urgencia y presión

- *"los horarios de entrevista son muy limitados"*
- *"dentro de las próximas 48 a 72 horas"*
- *"antes de que el sistema de programación se cierre"*

###  4. URL sospechosa en el mensaje

La URL `pablosanchez23fg.76evistatechno.com` es sospechosa porque:
- El subdominio `pablosanchez23fg` parece una cuenta personal
- El dominio `76evistatechno.com` no está relacionado con Indeed ni Cress-Sport
- No utiliza `https://` (protocolo seguro)

###  5. Código QR como técnica de evasión

El QR traslada a la víctima del ordenador al teléfono.

###  6. Proceso de entrevista atípico

| Proceso legítimo | Proceso fraudulento |
|------------------|---------------------|
| Zoom, Teams, Meet | App desconocida |
| Enlace directo | QR + redirecciones |
| Sin instalación | VPN + código de invitación |
| Sin acceso remoto | Solicitan control del dispositivo |

###  7. Beneficios inusuales en la oferta

"Uniforme proporcionado" para un puesto de **Especialista en Seguridad TI** es incompatible con el perfil.

---

##  Análisis del nombre "Theo Pierre Justin"

| Elemento | Análisis |
|----------|----------|
| **Nombre** | Theo Pierre Justin |
| **Posible origen** | Francés (coincide con CRESS Sport, empresa francesa) |
| **Vinculación con Cress-Sport** | No hay información pública |
| **Autenticidad** | No verificable; probablemente inventado |

---

##  Análisis de la empresa Cress-Sport IT

| Elemento | Realidad |
|----------|----------|
| **CRESS Sport (matriz)** | Empresa real fundada en 1995 en Talence (Francia), especializada en material deportivo |
| **Cress-Sport IT (filial)** | No existe registro público de esta filial en España |
| **Dirección** | Paseo de la Castellana 150, 28046 Madrid (ubicación real pero sin confirmación) |
| **Beneficios** | Incluye "uniforme" para un puesto IT (inconsistente) |

> **Conclusión:** La oferta fue creada por los estafadores utilizando el nombre de una empresa real para dar credibilidad.

---

##  Evidencias Recopiladas

| Evidencia | Descripción | Estado |
|-----------|-------------|--------|
| Mensaje en Indeed | Captura del mensaje en la bandeja de Indeed | ✅ Disponible |
| Metadatos del correo | `bounces.indeed.com`, `firmado por: indeed.com` | ✅ Disponible |
| URL de Indeed | `messages.indeed.com` (legítimo, estado 403) | ✅ Disponible |
| Análisis urlscan.io | Escaneo de `messages.indeed.com` | ✅ Disponible |
| Página QR | `improve-platform.com` (17 segundos) | ✅ Disponible |
| Análisis PowerDMARC | Índice 100/100 (NO detectado) | ✅ Disponible |
| Oferta de empleo | Publicada en Indeed para Cress-Sport IT | ✅ Disponible |

---

##  Medidas de Mitigación

### Para la víctima (aplicadas)

-  No se descargó ni instaló la aplicación
-  No se introdujo ningún código
-  No se compartió acceso al dispositivo
-  Se reportó a Indeed
-  Se documentó todo el proceso

### Recomendaciones para Indeed

1. **Verificar la identidad de los empleadores** antes de permitir el envío de mensajes a candidatos
2. **Implementar alertas** para mensajes que contengan enlaces sospechosos
3. **Revisar ofertas** de empresas que no tengan historial verificado
4. **Añadir advertencias** visibles en mensajes de empleadores no verificados

### Recomendaciones para candidatos

1. **Verificar siempre la empresa** antes de hacer clic en enlaces
2. **Desconfiar de las urgencias** - Las empresas serias no presionan así
3. **No descargar apps externas** - Las entrevistas usan Zoom, Teams o Meet
4. **Nunca compartir acceso remoto**
5. **No escanear QR de fuentes desconocidas**
6. **Verificar la oferta en la plataforma oficial**
7. **Contactar a la empresa por canales oficiales**

---

##  Lecciones Aprendidas

### Lo que hizo bien la víctima

- Detectó la inconsistencia en el proceso
- Reconoció las tácticas de presión
- Consultó antes de actuar
- Documentó todo el proceso
- No completó los pasos solicitados

### Lo que los atacantes hicieron "bien" (desde su perspectiva)

-  **Abusaron del sistema de mensajería de Indeed** (el mensaje parecía legítimo)
-  Suplantaron una empresa real (Cress-Sport IT)
-  Crearon urgencia
-  Usaron logos creíbles
-  Diseñaron flujo lógico
-  Explotaron una postulación real
-  Usaron QR como evasión
-  Usaron Cloudflare para ocultar su infraestructura
-  **Dominios NO catalogados como maliciosos**

---

##  Conclusiones

Este caso de phishing destaca por su **sofisticación sin precedentes** al combinar:

1. **Abuso del sistema de mensajería de Indeed** (mensaje enviado a través de la plataforma legítima)
2. **Suplantación de una empresa real** (Cress-Sport IT)
3. **Postulación real** de la víctima (expectativa)
4. **Nombre de reclutador inventado** ("Theo Pierre Justin")
5. **Tácticas de presión** para evitar el pensamiento crítico
6. **Código QR** como técnica de evasión
7. **Dominios de vida efímera** (17 segundos)
8. **Dominios NO catalogados** en bases de datos de amenazas
9. **Servicios de proxy** (Cloudflare) para ocultamiento

### La paradoja de la seguridad

> **El mensaje se envió a través del sistema LEGÍTIMO de Indeed.**  
> **El enlace de Indeed era LEGÍTIMO.**  
> **Y sin embargo, el contenido era FRAUDULENTO.**

Esto demuestra que:
- **Incluso las plataformas legítimas pueden ser abusadas**
- Un mensaje en la bandeja de Indeed **no garantiza su autenticidad**
- La confianza en la plataforma puede ser explotada
- **El factor humano sigue siendo la mejor defensa**

**La diferencia entre caer o no fue prestar atención a los detalles.**

---

##  Referencias

- [Indeed Centro de Seguridad](https://www.indeed.com/security)
- [Incibe - Phishing](https://www.incibe.es/protege-tu-empresa/amenazas/phishing)
- [urlscan.io](https://urlscan.io/)
- [PowerDMARC](https://powerdmarc.com/)
- [CRESS Sport (empresa real)](https://www.cress-sport.com/)

---

##  Indicadores de Compromiso (IOCs)

Disponibles en [iocs.txt](iocs.txt)

---

*Este writeup ha sido elaborado con fines educativos y de concienciación sobre ciberseguridad.*

**#Phishing #Cybersecurity #Indeed #CressSportIT #SecurityAwareness #JobScam #OSINT #SocialEngineering #Cloudflare #urlscan #PowerDMARC #PlatformAbuse**
