# Webtoon Downloader

<details>
<summary><b>ES Versión en Español (Haz clic para expandir)</b></summary>

<br>

Una aplicación de escritorio con interfaz gráfica diseñada para analizar, gestionar y descargar capítulos de series de Line Webtoon directamente a tu computadora. 

Esta herramienta permite descargar capítulos gratuitos, reclamar pases diarios, visualizar anuncios simulados para desbloquear contenido y utilizar tus monedas para descargar capítulos de pago de forma automática.

---

## ✨ Características Principales

* **Interfaz Gráfica Moderna:** Desarrollada con `customtkinter` (Modo Oscuro por defecto).
* **Gestión de Sesiones:** Inicia sesión con tu correo y contraseña. La sesión se guarda de forma segura en un archivo `config.json` para auto-logueo en usos futuros.
* **Detección Inteligente de Capítulos:**
  * Identifica capítulos gratuitos de acceso público.
  * Detecta capítulos comprados previamente (acceso infinito) o desbloqueos temporales.
  * Permite ver anuncios simulados (32 segundos de espera) para desbloquear capítulos con soporte para recompensas ("Reward Ads").
  * Compra automática de capítulos si tienes el saldo de monedas suficiente.
* **Selección Rápida por Rangos:** Escribe rangos (ej. `1-5, 8, 10-12`) para seleccionar masivamente qué descargar.
* **Organización Automática:** Los capítulos se guardan en una carpeta local llamada `Download/Nombre_de_la_Serie/Nombre_del_Capitulo/`.

---

## 📥 Instalación y Uso

### Opción A: Usar el Ejecutable (.exe) - *Recomendado*
Para facilitar el uso, no es necesario instalar Python ni librerías.
1. Ve a la pestaña de **Releases** a la derecha de este repositorio en GitHub.
2. Descarga el archivo `.exe` más reciente.
3. Guarda el ejecutable en una carpeta dedicada (por ejemplo, `C:\WebtoonDownloader\`), ya que el programa creará archivos adicionales (como `config.json` y la carpeta `Download/`) en la misma ubicación.
4. Abre el programa e inicia sesión.

### Opción B: Ejecutar desde el código fuente (`main.py`)
Si prefieres no usar el `.exe` y correr el script directamente en tu entorno Python:
1. Asegúrate de tener instalado **Python 3.8 o superior**.
2. Clona o descarga este repositorio en tu computadora.
3. Abre una terminal en la carpeta del proyecto e instala las dependencias necesarias ejecutando:
   `pip install requests pycryptodome customtkinter`
4. Ejecuta el programa con el siguiente comando:
   `python main.py`

### 🚀 Pasos para Descargar Capítulos
1. **Buscar Serie:** Pega la URL completa del Webtoon o simplemente el "ID" de la serie (ej: `2154`) en la barra de búsqueda y presiona **"Analizar"**.
2. **Seleccionar:** El panel izquierdo mostrará todos los capítulos y su estado (Gratis, Comprado, Temporal, Solo Ads, Pago). Puedes marcar las casillas manualmente o usar el cuadro de **"Selección Rápida"** a la derecha (escribiendo, por ejemplo, `1-10`).
3. **Descargar:** Presiona el botón verde **"📥 DESCARGAR"**.
4. En la consola inferior verás el progreso de la descarga en tiempo real. Si el capítulo requiere un anuncio, el programa esperará 32 segundos automáticamente. Las imágenes se guardarán en la carpeta `Download/` junto a tu script/ejecutable.

---

## 📱 Sobre Line Webtoon (App Oficial Global)

Este programa interactúa exclusivamente con los servidores de **Line Webtoon Global** (la versión occidental/global de la plataforma). 
Para que este programa funcione correctamente y puedas usar funciones como tus Monedas o Pases Diarios, necesitas tener una cuenta registrada en la plataforma oficial. 
Te recomendamos encarecidamente descargar la aplicación oficial en tu teléfono móvil para leer cómodamente, apoyar a los creadores de tus series favoritas dejándoles un "Me gusta", y gestionar tu cuenta:

* 🟢 **Android (Google Play):** [WEBTOON en Google Play](https://play.google.com/store/apps/details?id=com.naver.linewebtoon)
* 🍎 **iOS (App Store):** [WEBTOON en App Store](https://apps.apple.com/app/webtoon-comics/id894546091)
* 🌐 **Sitio Web Oficial:** [webtoons.com](https://www.webtoons.com/)

> **Nota:** Apoya siempre a los artistas originales leyendo los capítulos gratuitos en la app oficial y utilizando tus monedas legalmente.

---

## ⚙️ Explicación Técnica y APIs Utilizadas

El script principal (`main.py`) funciona simulando las peticiones que haría la aplicación oficial de Android de Webtoon. A continuación se explica el flujo técnico de las llamadas (APIs) utilizadas:

### 1. Sistema de Autenticación (`loginById`) y Encriptación
Webtoon no envía las contraseñas en texto plano. El script solicita primero unas llaves públicas (`n_value`, `e_value`, `session_key`) mediante el método `getRsaKey()`. Usando la librería `Crypto.Cipher.PKCS1_v1_5` (RSA), se cifra la combinación de correo, contraseña y llave de sesión, convirtiéndola a hexadecimal antes de enviarla mediante `POST`. Esto devuelve una cookie maestra llamada `NEO_SES`. 

Posteriormente, con `setDeviceInfo`, se registra el dispositivo virtual y se obtiene la cookie de verificación `NEO_CHK`.

### 2. Firmado de Peticiones HMAC-SHA1 (`get_signed_url`)
Para evitar que aplicaciones de terceros accedan a su API, Webtoon exige que cada petición vaya firmada criptográficamente. El script toma:
* La URL a consultar.
* El timestamp actual del servidor de Naver (`currentTime`).
* Una llave secreta de la API (`api_key`).
Con esto, genera un hash HMAC-SHA1 codificado en Base64 que se envía como los parámetros `msgpad` y `md` en absolutamente todas las peticiones `GET` o `POST`.

### 3. APIs de Análisis de Series
* **`titleInfo`**: Obtiene la información general de la serie, incluyendo su nombre real y la cantidad total de capítulos.
* **`productRightList` / `productRight`**: Verifica el "inventario" del usuario. Confirma si el usuario ya es dueño de un capítulo de pago (ya sea para siempre "infinito" o por alquiler "temporal").
* **`episodeList`**: Obtiene la lista base de los capítulos públicos.
* **`getProduct`**: Para los capítulos "Fast Pass" o Premium, esta API devuelve las opciones de compra. Detecta si el capítulo se puede pagar con Monedas, Pases Diarios (`DAILY_PASS`) o si acepta ver anuncios (`REWARD_AD`).

### 4. APIs de Compra y Recompensas
* **`buyProduct`**: Es la API que ejecuta la transacción. Se le envía el ID del producto y el tipo de venta (`complete`, `reward_ad`, `daily_pass`). Si es un anuncio, el programa simula la espera intencional de 32 segundos usando `time.sleep(32)` y solicita un token con `getImageSecureToken` antes de mandar el reclamo de éxito.

### 5. API de Descarga de Imágenes (`episodeInfoWithLogin` y `get_static_content`)
Una vez que el sistema verifica que el capítulo es gratuito o ha sido desbloqueado exitosamente en el paso anterior, se llama a `episodeInfoWithLogin`. Esta API devuelve un JSON con todas las URLs de las imágenes que componen el episodio. 
Finalmente, el script descarga cada imagen haciendo una petición al servidor de almacenamiento estático (`webtoon-phinf.pstatic.net`) utilizando un `Referer` específico (`http://m.webtoons.com/`) para evitar bloqueos por *Hotlinking*.

> **⚠️ Disclaimer:** Esta herramienta tiene fines exclusivamente educativos y de archivado personal. El abuso de las APIs o la descarga masiva puede resultar en el bloqueo de tu cuenta de Webtoon.

</details>

<details>
<summary><b>EN English Version (Click to expand)</b></summary>

<br>

A desktop GUI application designed to analyze, manage, and download Line Webtoon chapters directly to your computer.

This tool allows you to download free chapters, claim daily passes, watch simulated ads to unlock content, and automatically use your coins to download paid chapters.

---

## ✨ Key Features

* **Modern Graphical Interface:** Developed with `customtkinter` (Dark Mode by default).
* **Session Management:** Log in with your email and password. The session is securely saved in a `config.json` file for auto-login in future uses.
* **Smart Chapter Detection:**
  * Identifies free public access chapters.
  * Detects previously purchased chapters (infinite access) or temporary unlocks.
  * Allows watching simulated ads (32-second wait) to unlock chapters with reward support ("Reward Ads").
  * Automatic chapter purchase if you have enough coin balance.
* **Quick Range Selection:** Type ranges (e.g. `1-5, 8, 10-12`) to mass select what to download.
* **Automatic Organization:** Chapters are saved in a local folder named `Download/Series_Name/Chapter_Name/`.

---

## 📥 Installation and Usage

### Option A: Using the Executable (.exe) - *Recommended*
For ease of use, you do not need to install Python or any libraries.
1. Go to the **Releases** tab on the right side of this GitHub repository.
2. Download the latest `.exe` file.
3. Save the executable in a dedicated folder (for example, `C:\WebtoonDownloader\`), as the program will create additional files (like `config.json` and the `Download/` folder) in the same location.
4. Open the program and log in.

### Option B: Running from source (`main.py`)
If you prefer not to use the `.exe` and want to run the script directly in your Python environment:
1. Ensure you have **Python 3.8 or higher** installed.
2. Clone or download this repository to your computer.
3. Open a terminal in the project folder and install the required dependencies by running:
   `pip install requests pycryptodome customtkinter`
4. Run the program with the following command:
   `python main.py`

### 🚀 Steps to Download Chapters
1. **Search Series:** Paste the full Webtoon URL or simply the series "ID" (e.g. `2154`) in the search bar and press **"Analyze"**.
2. **Select:** The left panel will show all chapters and their status (Free, Purchased, Temporary, Ads Only, Paid). You can check the boxes manually or use the **"Quick Select"** box on the right (typing, for example, `1-10`).
3. **Download:** Press the green **"📥 DOWNLOAD"** button.
4. In the bottom console, you will see the download progress in real-time. If the chapter requires an ad, the program will wait 32 seconds automatically. The images will be saved in the `Download/` folder next to your script/executable.

---

## 📱 About Line Webtoon (Official Global App)

This program interacts exclusively with the **Line Webtoon Global** servers (the western/global version of the platform). 
For this program to work correctly and to use features like your Coins or Daily Passes, you must have a registered account on the official platform. 
We highly recommend downloading the official mobile app to read comfortably, support the creators of your favorite series by "liking" their episodes, and manage your account:

* 🟢 **Android (Google Play):** [WEBTOON on Google Play](https://play.google.com/store/apps/details?id=com.naver.linewebtoon)
* 🍎 **iOS (App Store):** [WEBTOON on App Store](https://apps.apple.com/app/webtoon-comics/id894546091)
* 🌐 **Official Website:** [webtoons.com](https://www.webtoons.com/)

> **Note:** Always support original artists by reading free chapters on the official app and using your coins legally.

---

## ⚙️ Technical Explanation and Used APIs

The main script (`main.py`) works by simulating the requests that the official Webtoon Android app would make. Below is the technical flow of the calls (APIs) used:

### 1. Authentication System (`loginById`) and Encryption
Webtoon does not send passwords in plain text. The script first requests public keys (`n_value`, `e_value`, `session_key`) using the `getRsaKey()` method. Using the `Crypto.Cipher.PKCS1_v1_5` (RSA) library, it encrypts the combination of email, password, and session key, converting it to hexadecimal before sending it via `POST`. This returns a master cookie called `NEO_SES`.

Later, with `setDeviceInfo`, the virtual device is registered, and the verification cookie `NEO_CHK` is obtained.

### 2. HMAC-SHA1 Request Signing (`get_signed_url`)
To prevent third-party apps from accessing their API, Webtoon requires every request to be cryptographically signed. The script takes:
* The URL to be queried.
* The current timestamp from the Naver server (`currentTime`).
* A secret API key (`api_key`).
With this, it generates a Base64 encoded HMAC-SHA1 hash that is sent as the `msgpad` and `md` parameters in absolutely all `GET` or `POST` requests.

### 3. Series Analysis APIs
* **`titleInfo`**: Gets the general information of the series, including its real name and the total number of chapters.
* **`productRightList` / `productRight`**: Checks the user's "inventory." It confirms if the user already owns a paid chapter (either forever "infinite" or by rental "temporary").
* **`episodeList`**: Gets the base list of public chapters.
* **`getProduct`**: For "Fast Pass" or Premium chapters, this API returns the purchase options. It detects if the chapter can be paid with Coins, Daily Passes (`DAILY_PASS`), or if it accepts watching ads (`REWARD_AD`).

### 4. Purchase and Rewards APIs
* **`buyProduct`**: This is the API that executes the transaction. It is sent the product ID and the sale type (`complete`, `reward_ad`, `daily_pass`). If it is an ad, the program simulates the intentional 32-second wait using `time.sleep(32)` and requests a token with `getImageSecureToken` before sending the success claim.

### 5. Image Download API (`episodeInfoWithLogin` and `get_static_content`)
Once the system verifies that the chapter is free or has been successfully unlocked in the previous step, `episodeInfoWithLogin` is called. This API returns a JSON with all the URLs of the images that make up the episode.
Finally, the script downloads each image by making a request to the static storage server (`webtoon-phinf.pstatic.net`) using a specific `Referer` (`http://m.webtoons.com/`) to avoid blocking by *Hotlinking*.

> **⚠️ Disclaimer:** This tool is strictly for educational purposes and personal archiving. Abuse of the APIs or mass downloading may result in the banning of your Webtoon account.

</details>
