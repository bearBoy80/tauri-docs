---
title: Modelo de procesos
sidebar:
  order: 0
i18nReady: true
---

Tauri emplea una arquitectura multiproceso similar a Electron o muchos navegadores web modernos. Esta guía explora las razones detrás de la elección del diseño y por qué es clave para escribir aplicaciones seguras.

## ¿Por qué múltiples procesos?

En los primeros días de las aplicaciones GUI, era común usar un solo proceso para realizar cálculos, dibujar la interfaz y reaccionar a la entrada del usuario. Como probablemente puedas adivinar, esto significaba que un cálculo costoso y de larga duración dejaría la interfaz de usuario sin respuesta, o peor aún, una falla en un componente de la aplicación provocaría el bloqueo de toda la aplicación.

Quedó claro que se necesitaba una arquitectura más resistente y las aplicaciones comenzaron a ejecutar diferentes componentes en diferentes procesos. Esto hace un uso mucho mejor de las CPU multinúcleo modernas y crea aplicaciones mucho más seguras. Un bloqueo en un componente ya no afecta a todo el sistema, ya que los componentes están aislados en diferentes procesos. Si un proceso entra en un estado inválido, podemos reiniciarlo fácilmente.

También podemos limitar el radio de explosión de posibles exploits entregando solo la cantidad mínima de permisos a cada proceso, lo suficiente para que puedan hacer su trabajo. Este patrón se conoce como el [Principio de privilegio mínimo], y lo ves en el mundo real todo el tiempo. Si viene un jardinero a podar tu seto, le das la llave de tu jardín. **No** les darías las llaves de tu casa; ¿por qué necesitarían acceso a eso? El mismo concepto se aplica a los programas informáticos. Cuanto menos acceso les demos, menos daño pueden hacer si se ven comprometidos.

## El proceso principal (Core Process)

Cada aplicación Tauri tiene un proceso principal, que actúa como el punto de entrada de la aplicación y que es el único componente con acceso total al sistema operativo.

La responsabilidad principal del Core es usar ese acceso para crear y orquestar ventanas de aplicaciones, menús de la bandeja del sistema o notificaciones. Tauri implementa las abstracciones multiplataforma necesarias para facilitar esto. También enruta toda la [Comunicación entre procesos] a través del proceso Core, lo que te permite interceptar, filtrar y manipular mensajes IPC en un lugar central.

El proceso Core también debe ser responsable de administrar el estado global, como la configuración o las conexiones de la base de datos. Esto te permite sincronizar fácilmente el estado entre ventanas y proteger tus datos confidenciales comerciales de miradas indiscretas en el Frontend.

Elegimos Rust para implementar Tauri debido a que su concepto de [Propiedad]
garantiza la seguridad de la memoria manteniendo un rendimiento excelente.

<figure>

```d2 sketch pad=50
direction: right

Core: {
  shape: diamond
}

"Events & Commands 1": {
  WebView1: WebView
}

"Events & Commands 2": {
  WebView2: WebView
}

"Events & Commands 3": {
  WebView3: WebView
}

Core -> "Events & Commands 1"{style.animated: true}
Core -> "Events & Commands 2"{style.animated: true}
Core -> "Events & Commands 3"{style.animated: true}

"Events & Commands 1" -> WebView1{style.animated: true}
"Events & Commands 2" -> WebView2{style.animated: true}
"Events & Commands 3" -> WebView3{style.animated: true}
```

<figcaption>Representación simplificada del modelo de procesos de Tauri. Un solo proceso Core administra uno o más procesos WebView.</figcaption>
</figure>

## El proceso WebView

El proceso Core no representa la interfaz de usuario (UI) real en sí misma; activa procesos WebView que aprovechan las bibliotecas WebView proporcionadas por el sistema operativo. Un WebView es un entorno similar a un navegador que ejecuta tu HTML, CSS y JavaScript.

Esto significa que la mayoría de tus técnicas y herramientas utilizadas en el desarrollo web tradicional se pueden utilizar para crear aplicaciones Tauri. Por ejemplo, muchos ejemplos de Tauri están escritos utilizando el marco frontend [Svelte] y el empaquetador [Vite].

Las mejores prácticas de seguridad también se aplican; por ejemplo, siempre debes desinfectar la entrada del usuario, nunca manejar secretos en el Frontend e idealmente diferir tanta lógica comercial como sea posible al proceso Core para mantener pequeña tu superficie de ataque.

A diferencia de otras soluciones similares, las bibliotecas WebView **no** se incluyen en tu ejecutable final, sino que se vinculan dinámicamente en tiempo de ejecución[^1]. Esto hace que tu aplicación sea _significativamente_ más pequeña, pero también significa que debes tener en cuenta las diferencias de plataforma, al igual que en el desarrollo web tradicional.

[^1]:
    Actualmente, Tauri utiliza [Microsoft Edge WebView2] en Windows, [WKWebView] en
    macOS y [webkitgtk] en Linux.

[principio de privilegio mínimo]: https://es.wikipedia.org/wiki/Principio_de_menor_privilegio
[comunicación entre procesos]: /es/concept/inter-process-communication/
[propiedad]: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
[microsoft edge webview2]: https://docs.microsoft.com/en-us/microsoft-edge/webview2/
[wkwebview]: https://developer.apple.com/documentation/webkit/wkwebview
[webkitgtk]: https://webkitgtk.org
[svelte]: https://svelte.dev/
[vite]: https://vitejs.dev/
