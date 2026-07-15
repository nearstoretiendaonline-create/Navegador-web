# Chrome-Lite Anonymous

Navegador web para Android, minimalista al estilo Chrome, con privacidad e
incógnito activados **por defecto** (no hay un modo normal que luego se
convierte en incógnito: esta es la única forma en que la app funciona).

## Qué incluye este esqueleto

| Área | Archivo | Qué hace |
|---|---|---|
| UI tipo Chrome | `res/layout/activity_main.xml` | Omnibox superior, barra de pestañas, ProgressBar, menú de 3 puntos |
| Modo oscuro nativo | `res/values-night/colors.xml` + `Theme.MaterialComponents.DayNight` | Sigue el tema del sistema automáticamente |
| Bloqueador de anuncios/rastreadores | `privacy/AdTrackerBlocker.kt` + `res/raw/blocklist.txt` | Intercepta a nivel de red en `shouldInterceptRequest`, antes de que la petición salga |
| HTTPS-Only | `privacy/PrivacyWebViewClient.kt` + `res/xml/network_security_config.xml` | Reescribe `http://` a `https://` y bloquea cleartext a nivel de OS |
| Spoofing de User-Agent | `privacy/UserAgentManager.kt` | UA genérico de escritorio, fijo por sesión (evitar fingerprinting sin romper sitios) |
| Sin historial/cookies persistentes | `privacy/PrivacyManager.kt` | `domStorageEnabled=false`, cookies de terceros bloqueadas, purga total al cerrar pestaña/app |
| Multipestaña | `core/TabManager.kt` | Un `WebView` aislado por pestaña |
| Build en la nube | `.github/workflows/build.yml`, `colab/build_in_colab.py` | Compila el APK sin Android Studio local |

## Decisiones de arquitectura ya tomadas (según tus respuestas)

- **Motor:** `android.webkit.WebView` (Chromium del sistema) — no GeckoView.
  Más liviano y sin dependencias extra que compilar.
- **Tipo de build:** solo APK de depuración (`assembleDebug`), sin firma ni
  keystore. Ideal para instalar en tu propio dispositivo vía `adb install`.
  Si más adelante quieres un release firmado, hay que añadir un keystore
  como GitHub Secret — te lo armo cuando lo necesites.

## Cómo compilar

### Opción A — GitHub Actions (recomendado, automático)
1. Sube esta carpeta como un repositorio nuevo en GitHub.
2. Cada `push` a `main` dispara `.github/workflows/build.yml`.
3. Ve a la pestaña **Actions** del repo → el run más reciente → descarga el
   artefacto `chrome-lite-anonymous-debug` (contiene el `.apk`).

### Opción B — Google Colab
1. Sube este repo a GitHub primero (Colab necesita clonarlo).
2. Abre `colab/Chrome_Lite_Anonymous_Build.ipynb` en Colab (o copia el
   contenido de `colab/build_in_colab.py` en una celda).
3. Edita `REPO_URL` con tu URL de GitHub.
4. Ejecuta todas las celdas — al final se descarga el `.apk` automáticamente
   a tu computadora.

### Instalar el APK
```
adb install app-debug.apk
```
o simplemente transfiere el `.apk` al teléfono y ábrelo (activa "instalar
de fuentes desconocidas" para esa app de origen).

## Ampliar el bloqueador de anuncios

`res/raw/blocklist.txt` trae ~50 dominios conocidos como punto de partida.
Para cobertura seria de producción, se recomienda fusionar con una lista
pública mantenida activamente, por ejemplo la lista de hosts de
StevenBlack (github.com/StevenBlack/hosts) o EasyList — basta con añadir
más líneas de dominio (uno por línea, sin `http://`) a ese mismo archivo.

## Próximos pasos sugeridos
- Firma de release + Play Integrity (si algún día se publica en Play Store).
- Un contador visible de "N rastreadores bloqueados" en la UI (el dato ya
  se calcula en `AdTrackerBlocker.blockedCount`, solo falta pintarlo).
- Soporte de "Buscar en la página" y descargas.
- Tests instrumentados para verificar que el bloqueo de cookies/HTTP
  realmente ocurre (JUnit + Espresso).
