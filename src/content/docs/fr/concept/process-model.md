---
title: Modèle de processus
sidebar:
  order: 0
i18nReady: true
---

Tauri utilise une architecture multi-processus similaire à Electron ou à de nombreux navigateurs Web modernes. Ce guide explore les raisons de ce choix de conception et pourquoi il est essentiel pour écrire des applications sécurisées.

## Pourquoi plusieurs processus ?

Au début des applications GUI, il était courant d'utiliser un seul processus pour effectuer des calculs, dessiner l'interface et réagir aux entrées de l'utilisateur. Comme vous pouvez probablement le deviner, cela signifiait qu'un calcul long et coûteux laisserait l'interface utilisateur insensible, ou pire, une défaillance dans un composant de l'application ferait planter toute l'application.

Il est devenu clair qu'une architecture plus résiliente était nécessaire, et les applications ont commencé à exécuter différents composants dans différents processus. Cela fait une bien meilleure utilisation des CPU multi-cœurs modernes et crée des applications beaucoup plus sûres. Un crash dans un composant n'affecte plus l'ensemble du système, car les composants sont isolés sur différents processus. Si un processus se retrouve dans un état invalide, nous pouvons facilement le redémarrer.

Nous pouvons également limiter le rayon d'explosion des exploits potentiels en ne distribuant que la quantité minimale de permissions à chaque processus, juste assez pour qu'ils puissent faire leur travail. Ce modèle est connu sous le nom de [Principe de moindre privilège], et vous le voyez tout le temps dans le monde réel. Si vous avez un jardinier qui vient tailler votre haie, vous lui donnez la clé de votre jardin. Vous ne lui donneriez **pas** les clés de votre maison ; pourquoi aurait-il besoin d'y accéder ? Le même concept s'applique aux programmes informatiques. Moins nous leur donnons d'accès, moins ils peuvent faire de mal s'ils sont compromis.

## Le processus principal (Core Process)

Chaque application Tauri a un processus principal, qui agit comme point d'entrée de l'application et qui est le seul composant ayant un accès complet au système d'exploitation.

La responsabilité principale du Core est d'utiliser cet accès pour créer et orchestrer des fenêtres d'application, des menus de la barre d'état système ou des notifications. Tauri implémente les abstractions multiplateformes nécessaires pour faciliter cela. Il achemine également toute la [Communication Inter-Processus] via le processus Core, vous permettant d'intercepter, de filtrer et de manipuler les messages IPC en un seul endroit central.

Le processus Core devrait également être responsable de la gestion de l'état global, comme les paramètres ou les connexions à la base de données. Cela vous permet de synchroniser facilement l'état entre les fenêtres et de protéger vos données commerciales sensibles des regards indiscrets dans le Frontend.

Nous avons choisi Rust pour implémenter Tauri car son concept de [Propriété]
garantit la sécurité de la mémoire tout en conservant d'excellentes performances.

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

<figcaption>Représentation simplifiée du modèle de processus Tauri. Un seul processus Core gère un ou plusieurs processus WebView.</figcaption>
</figure>

## Le processus WebView

Le processus Core ne rend pas l'interface utilisateur (UI) réelle elle-même ; il lance des processus WebView qui exploitent les bibliothèques WebView fournies par le système d'exploitation. Un WebView est un environnement de type navigateur qui exécute votre HTML, CSS et JavaScript.

Cela signifie que la plupart de vos techniques et outils utilisés dans le développement web traditionnel peuvent être utilisés pour créer des applications Tauri. Par exemple, de nombreux exemples Tauri sont écrits en utilisant le framework frontend [Svelte] et le bundler [Vite].

Les meilleures pratiques de sécurité s'appliquent également ; par exemple, vous devez toujours nettoyer les entrées utilisateur, ne jamais gérer de secrets dans le Frontend, et idéalement différer autant de logique métier que possible vers le processus Core pour garder votre surface d'attaque petite.

Contrairement à d'autres solutions similaires, les bibliothèques WebView ne sont **pas** incluses dans votre exécutable final mais liées dynamiquement au moment de l'exécution[^1]. Cela rend votre application _nettement_ plus petite, mais cela signifie également que vous devez garder à l'esprit les différences de plateforme, tout comme le développement web traditionnel.

[^1]:
    Actuellement, Tauri utilise [Microsoft Edge WebView2] sous Windows, [WKWebView] sous
    macOS et [webkitgtk] sous Linux.

[principe de moindre privilège]: https://fr.wikipedia.org/wiki/Principe_de_moindre_privil%C3%A8ge
[communication inter-processus]: /fr/concept/inter-process-communication/
[propriété]: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
[microsoft edge webview2]: https://docs.microsoft.com/en-us/microsoft-edge/webview2/
[wkwebview]: https://developer.apple.com/documentation/webkit/wkwebview
[webkitgtk]: https://webkitgtk.org
[svelte]: https://svelte.dev/
[vite]: https://vitejs.dev/
