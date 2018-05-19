# PWA GDG La Paz

## Preparando el Entorno

1. Instalamos Google Chrome
2. Instalamos [Ligthouse](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=es), como complemento de google
3. Instalamos [Web Server for Chrome](https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb?hl=en)
4. Instalamos Git
5. Instalamos Atom que usaremos como editor de código
6. [Descargamos los recursos](https://drive.google.com/file/d/1G5Q2G8AFYNQhAk2D6-fUKk73OUT5ysxi/view?usp=sharing)
7. Nos creamos una cuenta en [gihub](https://github.com/)

## Creamos nuestro entorno de desarrollo

1. Creamos el repositorio en github con el nombre **pwagdglapaz**
2. Clonamos el repositorio
3. Abrimos la carpeta con Atom

# Creamos nuestra PWA

## Creamos el archivo HTML `index.html` con una plantilla básica

1. Creamos el archivo HTML en la raiz
2. Creamos la estructura basica de HTML

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>PWA GDG La Paz</title>
  </head>
  <body>
    <h1>IO Extended 2018</h1>
  </body>
</html>
```
## Iniciamos el servidor y auditamos nuestra PWA

1. Ejecutamos **Web Server for Chrome**
2. Ejecutamos **lighthouse**

## Añadimos etiquetas meta a nuestro documento HTML
```html
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- Informa al navegador de que se trata de una PWA -->
    <meta name="mobile-web-app-capable" content="yes">
    <!-- Informa al navegador de iOS que se trata de una PWA -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <!-- Define el color del template -->
```

Volvemos a evaluar con **Ligthouse**

## Creamos nuestro archivo Manifest

1. Generamos nuestro archivo manifest en el siguiente [link](https://realfavicongenerator.net/)
2. Añadimos todos los iconos generados a nuestro proyecto
3. Modificamos nuestro archivo **site.webmanifest** con los datos que necesitemos

Volvemos a evaluar con **Ligthouse**

Añadimos el parametro start_url a nuestro archivo **site.webmanifest**
```
  "start_url": "/?utm_source=homescreen",
```
Volvemos a evaluar con **Ligthouse**

## Testear nuestra PWA

1. Google Chrome con la pestaña Application
2. Acceder desde un emulador de android
3. Acceder desde un dispositivo real

## Service Workers

* Creamos el archivo `app.js` dentro del directorio **js** para registrar nuestro service worker, a continuación el contenido del archivo:
```javascript
if ('serviceWorker' in navigator) {

  navigator.serviceWorker
    .register('./service-worker.js', { scope: './' })
    .then(function(registration) {
      console.log("Service Worker Registered");
    })
    .catch(function(err) {
      console.log("Service Worker Failed to Register", err);
    })

}
```
* Creamos el archivo `service-worker.js` en la raiz de nuestro proyecto y añadimos el siguiente contenido:
```javascript
// Set a name for the current cache
var cacheName = 'v1';

// Default files to always cache
var cacheFiles = [
	'./',
	'./index.html',
	'./js/app.js'
]


self.addEventListener('install', function(e) {
    console.log('[ServiceWorker] Installed');

    // e.waitUntil Delays the event until the Promise is resolved
    e.waitUntil(

    	// Open the cache
	    caches.open(cacheName).then(function(cache) {

	    	// Add all the default files to the cache
			console.log('[ServiceWorker] Caching cacheFiles');
			return cache.addAll(cacheFiles);
	    })
	); // end e.waitUntil
});


self.addEventListener('activate', function(e) {
    console.log('[ServiceWorker] Activated');

    e.waitUntil(

    	// Get all the cache keys (cacheName)
		caches.keys().then(function(cacheNames) {
			return Promise.all(cacheNames.map(function(thisCacheName) {

				// If a cached item is saved under a previous cacheName
				if (thisCacheName !== cacheName) {

					// Delete that cached file
					console.log('[ServiceWorker] Removing Cached Files from Cache - ', thisCacheName);
					return caches.delete(thisCacheName);
				}
			}));
		})
	); // end e.waitUntil

});


self.addEventListener('fetch', function(e) {
	console.log('[ServiceWorker] Fetch', e.request.url);

	// e.respondWidth Responds to the fetch event
	e.respondWith(

		// Check in cache for the request being made
		caches.match(e.request)


			.then(function(response) {

				// If the request is in the cache
				if ( response ) {
					console.log("[ServiceWorker] Found in Cache", e.request.url, response);
					// Return the cached version
					return response;
				}

				// If the request is NOT in the cache, fetch and cache

				var requestClone = e.request.clone();
				fetch(requestClone)
					.then(function(response) {

						if ( !response ) {
							console.log("[ServiceWorker] No response from fetch ")
							return response;
						}

						var responseClone = response.clone();

						//  Open the cache
						caches.open(cacheName).then(function(cache) {

							// Put the fetched response in the cache
							cache.put(e.request, responseClone);
							console.log('[ServiceWorker] New Data Cached', e.request.url);

							// Return the response
							return response;

				        }); // end caches.open

					})
					.catch(function(err) {
						console.log('[ServiceWorker] Error Fetching & Caching New Data', err);
					});


			}) // end caches.match(e.request)
	); // end e.respondWith
});
```

* Enlazamos nuestro documento HTML con el archivo `app.js` usando el siguiente código
```html
<script src="js/app.js"></script>
```
* Analizamos con **Ligthouse**
