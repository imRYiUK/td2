# Boutique Diayma

## Bugs Corrigés

### Bug #1 : Calcul incorrect du total du panier
**Problème** : La méthode `GetTotalValue()` dans `Cart.cs` ne multipliait pas le prix par la quantité.
```csharp
// Avant (incorrect)
return GetCartLineList().Sum(x => x.Product.Price);

// Après (corrigé)
return GetCartLineList().Sum(x => x.Product.Price * x.Quantity);
```
**Impact** : Le total du panier était incorrect lorsqu'un produit avait une quantité supérieure à 1.

### Bug #2 : Calcul incorrect de la moyenne du panier
**Problème** : La méthode `GetAverageValue()` dans `Cart.cs` calculait la moyenne des prix sans tenir compte des quantités.
```csharp
// Avant (incorrect)
return GetCartLineList().Average(x => x.Product.Price);

// Après (corrigé)
var totalQuantity = GetCartLineList().Sum(x => x.Quantity);
if (totalQuantity > 0)
    return GetTotalValue() / totalQuantity;
else
    return 0.0;
```
**Impact** : La moyenne du panier ne reflétait pas le prix moyen par article en tenant compte des quantités.

---

## Analyse du Flux d'Exécution - Affichage de la Page d'Accueil

### Points d'Arrêt Définis
- **a)** `CartSummaryViewComponent` ligne 12 (constructeur)
- **b)** `ProductController` ligne 15 (constructeur)
- **c)** `OrderController` ligne 17 (constructeur)
- **d)** `CartController` ligne 15 (constructeur)
- **e)** `Startup` ligne 20 (ConfigureServices)

### Flux d'Exécution lors de l'Affichage de la Page d'Accueil

Lorsque l'utilisateur accède à la page d'accueil de l'application (route par défaut : `/Product/Index`), voici le flux d'exécution détaillé :

#### 1. **Démarrage de l'Application**

**Namespace** : `P2FixAnAppDotNetCode`  
**Classe** : `Program`  
**Méthode** : `Main(string[] args)`
- Point d'entrée de l'application
- Appelle `BuildWebHost(args).Run()`

**Méthode** : `BuildWebHost(string[] args)`
- Configure le serveur web avec `WebHost.CreateDefaultBuilder(args)`
- Spécifie la classe `Startup` pour la configuration

---

#### 2. **Configuration des Services (Point d'arrêt e)**

**Namespace** : `P2FixAnAppDotNetCode`  
**Classe** : `Startup`  
**Méthode** : `ConfigureServices(IServiceCollection services)` - **Ligne 20**
- **Mode de débogage recommandé** : **Pas à pas principal** (Step Over)
- Configure l'injection de dépendances :
  - `ICart` → `Cart` (Singleton)
  - `IProductService` → `ProductService` (Transient)
  - `IProductRepository` → `ProductRepository` (Transient)
  - Localisation, Session, MVC, etc.

**Méthode** : `Configure(IApplicationBuilder app, IHostingEnvironment env)`
- Configure le pipeline de requêtes HTTP
- Configure le routage : route par défaut = `{controller=Product}/{action=Index}/{id?}`

---

#### 3. **Réception de la Requête HTTP GET /**

Le middleware ASP.NET Core route la requête vers le contrôleur par défaut : `ProductController.Index()`

---

#### 4. **Injection de Dépendances et Création du ProductController (Point d'arrêt b)**

**Namespace** : `P2FixAnAppDotNetCode.Controllers`  
**Classe** : `ProductController`  
**Méthode** : `ProductController(IProductService productService, ILanguageService languageService)` - **Ligne 15**
- **Mode de débogage recommandé** : **Pas à pas détaillé** (Step Into)
- Le conteneur d'injection de dépendances crée une instance de `ProductService`
- Injection de `IProductService` et `ILanguageService`

---

#### 5. **Création du ProductService**

**Namespace** : `P2FixAnAppDotNetCode.Models.Services`  
**Classe** : `ProductService`  
**Méthode** : `ProductService(IProductRepository productRepository, IOrderRepository orderRepository)`
- **Mode de débogage recommandé** : **Pas à pas détaillé** (Step Into)
- Injection de `IProductRepository` et `IOrderRepository`

---

#### 6. **Création du ProductRepository**

**Namespace** : `P2FixAnAppDotNetCode.Models.Repositories`  
**Classe** : `ProductRepository`  
**Méthode** : `ProductRepository()`
- **Mode de débogage recommandé** : **Pas à pas détaillé** (Step Into)
- Initialise la liste `_products`
- Appelle `GenerateProductData()`

**Méthode** : `GenerateProductData()`
- **Mode de débogage recommandé** : **Pas à pas principal** (Step Over)
- Génère les 9 produits par défaut (XIAOMI, Micro-ondes, Casque, etc.)

---

#### 7. **Exécution de l'Action Index du ProductController**

**Namespace** : `P2FixAnAppDotNetCode.Controllers`  
**Classe** : `ProductController`  
**Méthode** : `Index()`
- **Mode de débogage recommandé** : **Pas à pas détaillé** (Step Into)
- Appelle `_productService.GetAllProducts()`

---

#### 8. **Récupération des Produits**

**Namespace** : `P2FixAnAppDotNetCode.Models.Services`  
**Classe** : `ProductService`  
**Méthode** : `GetAllProducts()`
- **Mode de débogage recommandé** : **Pas à pas détaillé** (Step Into)
- Appelle `_productRepository.GetAllProducts().ToList()`

**Namespace** : `P2FixAnAppDotNetCode.Models.Repositories`  
**Classe** : `ProductRepository`  
**Méthode** : `GetAllProducts()`
- **Mode de débogage recommandé** : **Pas à pas principal** (Step Over)
- Filtre les produits avec stock > 0
- Trie par nom
- Retourne un tableau de produits

---

#### 9. **Retour de la Vue**

**Namespace** : `P2FixAnAppDotNetCode.Controllers`  
**Classe** : `ProductController`  
**Méthode** : `Index()`
- Retourne `View(products)` avec la liste des produits
- Le moteur Razor cherche la vue `/Views/Product/Index.cshtml`

---

#### 10. **Rendu de la Vue avec Layout**

**Vue** : `/Views/Product/Index.cshtml`
- **Mode de débogage recommandé** : **Pas à pas principal** (Step Over)
- Utilise le layout `_Layout.cshtml`
- Affiche la table des produits

**Vue** : `/Views/Shared/_Layout.cshtml`
- Contient la structure HTML commune
- **Ligne 21** : Appelle `@await Component.InvokeAsync("CartSummary")`

---

#### 11. **Invocation du CartSummaryViewComponent (Point d'arrêt a)**

**Namespace** : `P2FixAnAppDotNetCode.Components`  
**Classe** : `CartSummaryViewComponent`  
**Méthode** : `CartSummaryViewComponent(ICart cart)` - **Ligne 12**
- **Mode de débogage recommandé** : **Pas à pas détaillé** (Step Into)
- Injection du panier (singleton `Cart`)
- Cast de `ICart` vers `Cart`

**Méthode** : `Invoke()`
- **Mode de débogage recommandé** : **Pas à pas détaillé** (Step Into)
- Retourne `View(_cart)` pour afficher le résumé du panier dans la navbar

---

#### 12. **Rendu Final**

Le HTML complet est généré et envoyé au navigateur, affichant :
- La navbar avec le nom de la boutique et le résumé du panier
- La liste des produits dans un tableau
- Le footer avec le sélecteur de langue

---

### Résumé des Namespaces, Classes et Méthodes Visités (dans l'ordre)

1. **P2FixAnAppDotNetCode.Program**
   - `Main(string[] args)`
   - `BuildWebHost(string[] args)`

2. **P2FixAnAppDotNetCode.Startup** ⚡ *Point d'arrêt e - ligne 20*
   - `ConfigureServices(IServiceCollection services)`
   - `Configure(IApplicationBuilder app, IHostingEnvironment env)`

3. **P2FixAnAppDotNetCode.Models.Repositories.ProductRepository**
   - `ProductRepository()` (constructeur)
   - `GenerateProductData()`

4. **P2FixAnAppDotNetCode.Models.Services.ProductService**
   - `ProductService(IProductRepository, IOrderRepository)` (constructeur)

5. **P2FixAnAppDotNetCode.Controllers.ProductController** ⚡ *Point d'arrêt b - ligne 15*
   - `ProductController(IProductService, ILanguageService)` (constructeur)
   - `Index()`

6. **P2FixAnAppDotNetCode.Models.Services.ProductService**
   - `GetAllProducts()`

7. **P2FixAnAppDotNetCode.Models.Repositories.ProductRepository**
   - `GetAllProducts()`

8. **Vue Razor** : `/Views/Product/Index.cshtml`

9. **Vue Razor** : `/Views/Shared/_Layout.cshtml`

10. **P2FixAnAppDotNetCode.Components.CartSummaryViewComponent** ⚡ *Point d'arrêt a - ligne 12*
    - `CartSummaryViewComponent(ICart cart)` (constructeur)
    - `Invoke()`

---

### Notes sur les Points d'Arrêt Non Visités

- **Point d'arrêt c** (`OrderController` ligne 17) : Non visité lors de l'affichage de la page d'accueil. Ce contrôleur est utilisé uniquement lors de la création d'une commande.

- **Point d'arrêt d** (`CartController` ligne 15) : Non visité lors de l'affichage de la page d'accueil. Ce contrôleur est utilisé pour ajouter/supprimer des produits du panier ou afficher le panier.

---

### Modes de Débogage Recommandés

- **Pas à pas détaillé (Step Into - F11)** : Pour suivre l'exécution dans les méthodes appelées
  - Constructeurs des contrôleurs
  - Méthodes d'action (`Index()`)
  - Appels de services (`GetAllProducts()`)
  - ViewComponents (`Invoke()`)

- **Pas à pas principal (Step Over - F10)** : Pour exécuter une ligne sans entrer dans les détails
  - Configuration dans `Startup`
  - Génération de données (`GenerateProductData()`)
  - Opérations LINQ simples

- **Pas à pas sortant (Step Out - Shift+F11)** : Pour sortir rapidement d'une méthode et revenir à l'appelant
  - Utile si on entre accidentellement dans du code framework
  - Pour remonter rapidement dans la pile d'appels


---

## Déploiement Windows

### Publication de l'Application

L'application a été publiée en tant qu'exécutable Windows autonome (self-contained) avec le runtime .NET Core 2.0 inclus.

**Commande de publication :**
```bash
dotnet publish -c Release -r win-x64 --self-contained true
```

### Emplacement de l'Exécutable

Le package de déploiement se trouve dans :
```
P2FixAnAppDotNetCode/bin/Release/netcoreapp2.0/win-x64/publish/
```

**Fichiers principaux :**
- `Diayma.exe` (77 KB) - Exécutable principal de l'application
- `Diayma.dll` (21 KB) - Bibliothèque de l'application
- `Diayma.deps.json` (378 KB) - Dépendances de l'application
- `Diayma.runtimeconfig.json` - Configuration du runtime
- Runtime .NET Core complet (~102 MB au total)

### Instructions de Déploiement

1. **Copier le dossier complet** `publish/` sur une machine Windows
2. **Aucune installation de .NET Core requise** - Le runtime est inclus
3. **Lancer l'application** en double-cliquant sur `Diayma.exe`
4. **Accéder à l'application** via le navigateur à l'adresse affichée dans la console (généralement `http://localhost:5000`)

### Configuration du Port (Optionnel)

Pour modifier le port d'écoute, vous pouvez :
- Utiliser la variable d'environnement : `set ASPNETCORE_URLS=http://localhost:8080`
- Ou lancer avec : `Diayma.exe --urls "http://localhost:8080"`

### Notes Importantes

- **Taille du package** : ~102 MB (runtime inclus pour fonctionnement autonome)
- **Plateforme cible** : Windows x64
- **Aucune dépendance externe** : Tout est inclus dans le dossier publish
- **Pare-feu** : Peut nécessiter une autorisation pour écouter sur un port HTTP

### Création d'une Archive de Déploiement

Pour faciliter le transfert, vous pouvez créer une archive :
```bash
cd P2FixAnAppDotNetCode/bin/Release/netcoreapp2.0/win-x64/
zip -r Diayma-Windows-x64.zip publish/
```

Ou avec tar :
```bash
tar -czf Diayma-Windows-x64.tar.gz publish/
```
