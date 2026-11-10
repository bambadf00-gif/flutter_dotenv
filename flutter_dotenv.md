# 📘 Flutter : apprendre à utiliser flutter_dotenv

## 1. Qu'est-ce que flutter_dotenv ?

`flutter_dotenv` est un package Flutter qui permet de lire des variables d'environnement depuis un fichier `.env`.

Il est notamment utilisé pour centraliser les configurations d'une application, comme :

- Les clés API.
- Les tokens d'authentification.
- Les URL de services.
- Les paramètres de configuration.

**Son rôle principal :** faciliter la gestion des configurations sans avoir à écrire directement les valeurs dans le code Dart.

---

## 2. Installer le package

Dans le terminal, à la racine du projet Flutter, exécuter :

```bash
flutter pub add flutter_dotenv
```

Cette commande installe le package et l'ajoute aux dépendances du projet dans `pubspec.yaml`.

---

## 3. Créer le fichier .env

À la racine du projet, au même niveau que `pubspec.yaml`, créer un fichier nommé :

```text
.env
```

Exemple de contenu :

```env
TMDB_API_KEY=ma_cle_api
TMDB_READ_ACCESS_TOKEN=mon_token
```

### À quoi sert ce fichier ?

Il permet de centraliser les valeurs de configuration de l'application.

Au lieu d'écrire une clé API directement dans un fichier Dart, on la conserve dans `.env`.

---

## 4. Déclarer le fichier .env dans pubspec.yaml

Flutter doit savoir que `.env` est une ressource à inclure dans les assets de l'application.

Dans `pubspec.yaml`, ajouter `.env` dans la section `flutter` :

```yaml
flutter:
  uses-material-design: true

  assets:
    - .env
```

**Attention :** si une section `assets` existe déjà, ajouter simplement `- .env` dedans. Ne pas créer une deuxième section `flutter`.

### Pourquoi cette étape ?

`flutter_dotenv` utilise les assets Flutter pour accéder au fichier `.env`.

Sans cette déclaration, le fichier risque de ne pas être trouvé au chargement.

---

## 5. Charger le fichier .env dans main.dart

Dans `lib/main.dart`, importer le package :

```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';
```

Puis charger le fichier avant de démarrer l'application :

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await dotenv.load();

  runApp(const MyApp());
}
```

### Explication

- `WidgetsFlutterBinding.ensureInitialized()` : initialise les services Flutter nécessaires avant de lancer l'application.
- `dotenv.load()` : lit le fichier `.env`.
- `await` : attend que le chargement soit terminé avant de continuer.
- `runApp()` : démarre l'application.

**Important :** on charge le fichier une seule fois au démarrage, pas à chaque requête API.

---

## 6. Récupérer les variables du fichier .env

Pour récupérer une variable, utiliser :

```dart
dotenv.env['NOM_DE_LA_VARIABLE']
```

Exemple :

```dart
final token = dotenv.env['TMDB_READ_ACCESS_TOKEN'];
```

La valeur récupérée est une chaîne de caractères nullable (`String?`).

### Comprendre l'opérateur ??

On peut utiliser `??` pour fournir une valeur par défaut si la variable est absente :

```dart
final token = dotenv.env['TMDB_READ_ACCESS_TOKEN'] ?? '';
```

Cela signifie :

- Si la variable existe, récupérer sa valeur.
- Si elle vaut `null`, retourner une chaîne vide (`''`).

---

## 7. Créer un fichier .env.example

À la racine du projet, créer :

```text
.env.example
```

Exemple :

```env
TMDB_API_KEY=your_api_key
TMDB_READ_ACCESS_TOKEN=your_read_access_token
```

### À quoi sert ce fichier ?

Il sert de modèle aux autres développeurs.

Lorsqu'une personne récupère ton projet depuis GitHub :

1. Elle copie `.env.example`.
2. Elle renomme la copie en `.env`.
3. Elle remplace les valeurs fictives par ses propres clés.

Ne jamais mettre les vraies clés dans `.env.example`.

---

## 8. Protéger le fichier .env avec .gitignore

Dans le fichier `.gitignore`, ajouter :

```gitignore
.env
.env.*
!.env.example
```

### Explication

- `.env` : ignore le fichier contenant les vraies configurations.
- `.env.*` : ignore les variantes comme `.env.local` ou `.env.production`.
- `!.env.example` : autorise l'envoi du modèle `.env.example` sur GitHub.

Ainsi, les autres développeurs peuvent connaître les variables nécessaires sans récupérer tes valeurs personnelles.

---

## 9. Tester le chargement

Pour vérifier que le fichier est correctement chargé, ajouter temporairement dans `main.dart`, après `dotenv.load()` :

```dart
print(
  'Token chargé : ${dotenv.env['TMDB_READ_ACCESS_TOKEN']?.isNotEmpty}',
);
```

Si le résultat est `true`, cela signifie que la variable existe et contient une valeur.

Supprimer ensuite ce test.

---

## 10. Bonne pratique : créer un ApiConfig

Jusqu'ici, nous avons vu comment utiliser `flutter_dotenv` directement pour récupérer les variables du fichier `.env`.

Cependant, dans un projet bien organisé, il est préférable de ne pas répéter partout les mêmes expressions :

```dart
dotenv.env['TMDB_READ_ACCESS_TOKEN']
```

Pour cela, nous pouvons créer une classe appelée `ApiConfig`.

**Attention : ApiConfig ne fait pas partie du package flutter_dotenv.** C'est une classe Dart que nous créons nous-mêmes pour mieux organiser notre code.

### Où la créer ?

Dans notre architecture Flutter :

```text
lib/
  core/
    config/
      api_config.dart
```

### Exemple de code

Dans `api_config.dart` :

```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';

class ApiConfig {
  static const String baseUrl = 'https://api.themoviedb.org/3';

  static String get apiKey =>
      dotenv.env['TMDB_API_KEY'] ?? '';

  static String get token =>
      dotenv.env['TMDB_READ_ACCESS_TOKEN'] ?? '';
}
```

### Pourquoi utiliser ApiConfig ?

Au lieu d'écrire :

```dart
dotenv.env['TMDB_READ_ACCESS_TOKEN']
```

Dans plusieurs fichiers, nous pouvons simplement écrire :

```dart
ApiConfig.token
```

De même, au lieu de répéter l'URL de l'API :

```dart
ApiConfig.baseUrl
```

### Les avantages

- Centraliser les configurations de l'API dans un seul fichier.
- Éviter de répéter les mêmes expressions.
- Faciliter les modifications futures.
- Rendre le code plus lisible et plus facile à maintenir.

**À retenir :** ApiConfig est facultatif. `flutter_dotenv` fonctionne parfaitement sans lui. Mais pour une bonne pratique d'architecture, il est conseillé de centraliser les configurations dans une classe dédiée, surtout lorsque le projet grandit.

---

## 11. Comprendre le fonctionnement général

Voici comment les différents éléments travaillent ensemble :

```text
       Fichier .env
            |
            v
       pubspec.yaml
    (déclaration en asset)
            |
            v
        main.dart
     (dotenv.load())
            |
            v
       flutter_dotenv
       (lecture du fichier)
            |
            v
       ApiConfig
    (centralisation des valeurs)
            |
            v
       ApiClient + Dio
    (configuration des requêtes)
            |
            v
          TMDB
```

---

## 12. Récapitulatif des étapes

| Étape | Action | Utilité |
|---|---|---|
| 1 | Installer le package | Ajouter flutter_dotenv |
| 2 | Créer `.env` | Stocker les configurations |
| 3 | Modifier `pubspec.yaml` | Déclarer le fichier en asset |
| 4 | Charger dans `main.dart` | Lire le fichier au démarrage |
| 5 | Utiliser `dotenv.env` | Récupérer les variables |
| 6 | Créer `.env.example` | Fournir un modèle |
| 7 | Modifier `.gitignore` | Éviter de publier `.env` |
| 8 | Créer `ApiConfig` (facultatif) | Centraliser les configurations |

---

## 13. ⚠️ Important : sécurité

Un fichier `.env` intégré aux assets d'une application Flutter peut être extrait de l'application compilée.

Par conséquent :

- Ne considère pas les clés présentes dans `.env` comme secrètes une fois l'application distribuée.
- Ne publie jamais tes vraies clés sur GitHub.
- Pour protéger réellement un token privé dans une application publique, utilise un backend qui conserve ce token côté serveur.

`.gitignore` empêche les commits accidentels, mais ne protège pas le contenu du fichier dans l'application compilée.

---

## Conclusion

`flutter_dotenv` permet de séparer les configurations du code source de l'application.

Les éléments essentiels à retenir sont :

- `dotenv.load()` : charge le fichier `.env`.
- `dotenv.env['NOM']` : récupère une variable.
- `?? ''` : fournit une valeur par défaut si la variable est absente.
- `ApiConfig` : permet de centraliser les configurations pour avoir un code mieux organisé.

C'est un outil pratique pour organiser les configurations de son projet Flutter.