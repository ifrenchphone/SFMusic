# SFMusic

Un lecteur de musique pour **iOS 6**, qui lit directement une bibliothèque hébergée sur un serveur **SFTP**. Compilé depuis Linux, sans Xcode et sans Mac.

L'idée de départ : un iPhone 5 sous iOS 6.1.4 n'a plus accès à grand-chose, mais il a un Wi-Fi, un DAC et un écran. Il suffit d'une app qui sache parler SSH pour qu'il redevienne un lecteur réseau tout à fait convenable.

## Ce que ça fait

- **Lecture depuis SFTP** — connexion par mot de passe, navigation dans l'arborescence distante, téléchargement en cache puis lecture locale (pas de streaming : plus tolérant à un Wi-Fi capricieux)
- **Bibliothèque indexée** — albums, artistes, morceaux, avec pochettes générées à la volée (couleur dérivée du nom, donc stable d'un lancement à l'autre)
- **Playlists** — création, réordonnancement, import `.m3u`, ajout d'un morceau ou d'un dossier entier d'un seul geste, y compris récursivement
- **Explorateur de dossiers** — arborescence dépliable en place, chargement paresseux, choix de la racine
- **Lecture en arrière-plan** — le son continue sur le SpringBoard et dans les autres apps, avec contrôles distants et écran verrouillé
- **Reprise** — position et file d'attente restaurées au lancement suivant
- **Diagnostic intégré** — état de la session audio, cycle de vie, dernier passage en arrière-plan, envoi du rapport sur le serveur

Interface dessinée pour l'époque : thème sombre par défaut, avec une variante métal brossé clair derrière un seul interrupteur (`Theme.isDark`). Aucune API postérieure à iOS 6 n'est utilisée, et la compilation échoue si l'une s'y glisse.

## Construire sans Xcode

C'est la partie qui a demandé le plus de travail. La chaîne complète tourne sur Linux :

- **clang** pour produire des objets Mach-O armv7
- **ld64** de [cctools-port](https://github.com/tpoechtrager/cctools-port) pour l'édition de liens — le backend Mach-O de `lld` ne gère pas armv7
- un **convertisseur `.tbd` → dylibs stub** maison, parce que ce `ld64` est compilé sans libtapi et ne sait donc pas lire les stubs texte d'un SDK moderne
- **libssh2 1.11** + **mbedTLS 3.6** compilés en statique pour armv7
- **ldid** pour une pseudo-signature ad-hoc en SHA-1 (iOS 6 ne comprend pas les CodeDirectory SHA-256)

```sh
./tools/setup_toolchain.sh   # une fois
./build.sh                   # -> build/SFMusic.app
./tools/verify.sh            # contrôles avant installation
./tools/make_deb.sh 1.0-1    # paquet Cydia, optionnel
```

`verify.sh` vérifie le bundle, les tailles d'icônes, l'en-tête Mach-O, la version minimale, les versions de bibliothèques liées, la signature, et qu'aucune classe postérieure à iOS 6 n'est référencée.

## Installation

Appareil jailbreaké, avec AppSync Unified pour accepter un binaire non signé.

```sh
# .ipa
ipainstaller SFMusic-unsigned.ipa

# ou .app
unzip SFMusic.app.zip -d /Applications/
chown -R mobile:mobile /Applications/SFMusic.app
chmod 755 /Applications/SFMusic.app/SFMusic
su mobile -c uicache
```

## Notes techniques

Deux pièges valent d'être signalés, parce qu'ils ne se manifestent que hors d'Xcode et qu'aucun des deux ne produit d'erreur.

**Les versions de bibliothèques liées.** Le générateur de stubs inscrivait `1.0.0` comme *current version* dans chaque `LC_LOAD_DYLIB`. dyld ne contrôle que la *compatibility version*, donc tout se chargeait normalement — mais `NSVersionOfLinkTimeLibrary()` renvoie la *current version*, et c'est ce que lisent UIKit et Foundation pour dater une application. Un binaire lié à « libSystem 1.0 » paraît antérieur à iPhone OS 2, donc antérieur au multitâche : iOS lui applique le cycle de vie des vieilles apps, c'est-à-dire `applicationDidEnterBackground:` suivi d'une terminaison immédiate, quoi que déclare `UIBackgroundModes`. Le binaire porte désormais des versions d'époque iOS 6.1, et `verify.sh` refuse toute compilation qui retomberait à `1.0.0`.

**Les `UIBackgroundTask`.** Une chaîne de tâches renouvelées depuis leur propre gestionnaire d'expiration et jamais soldées fait terminer l'application par iOS — le mode audio n'en exempte pas. La lecture en arrière-plan ne repose donc que sur l'assertion audio, et la seule tâche restante entoure un téléchargement, toujours refermée.

Le reste est du terrain connu mais daté : les tables groupées d'iOS 6 imposent un fond blanc arrondi qui écrase `cell.backgroundColor`, `UITabBar.barStyle` et `UITableViewCellSelectionStyleDefault` n'existent qu'à partir d'iOS 7, et `clock_gettime` n'apparaît qu'en iOS 10 — d'où `MBEDTLS_PLATFORM_MS_TIME_ALT` dans la configuration de mbedTLS.

## Limites

- armv7 uniquement, iOS 6.0 minimum
- authentification SFTP par mot de passe (pas de clé)
- lecture après téléchargement complet, pas de streaming progressif
- pas de lecture des tags ID3 : les métadonnées viennent des noms de fichiers et de dossiers
- appareil jailbreaké requis, l'application n'étant pas signée par Apple
