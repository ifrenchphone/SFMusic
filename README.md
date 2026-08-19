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

## Notes techniques

Deux pièges valent d'être signalés, parce qu'ils ne se manifestent que hors d'Xcode et qu'aucun des deux ne produit d'erreur.

## Limites

- armv7 uniquement, iOS 6.0 minimum
- authentification SFTP par mot de passe (pas de clé)
- lecture après téléchargement complet, pas de streaming progressif
- pas de lecture des tags ID3 : les métadonnées viennent des noms de fichiers et de dossiers
- appareil jailbreaké requis, l'application n'étant pas signée par Apple
