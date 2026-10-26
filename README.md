# SèchePro — Journal nutritionnel

Application web installable sur téléphone et ordinateur, adaptée aux petits écrans et utilisable hors ligne après son premier chargement. GitHub Pages peut la publier directement.

## Fonctionnalités

- Journal visuel des repas, jauge de calories sur l’accueil et suivi des macronutriments
- Total d’hydratation et jauge visibles sur l’accueil ; saisie rapide des quantités, correction du total et réglage de l’objectif dans Nutrition
- Profil et objectifs personnalisables : le poids actuel remplace le nombre de séances par semaine dans les champs principaux (fréquence des séances conservée dans les réglages avancés)
- Suivi des activités, des pas, des séances de musculation (séries, records et progression) et du poids (courbe et moyennes hebdomadaires)
- Onglet Séance : sélection de plusieurs groupes musculaires, propositions d’exercices illustrées par des schémas et préparation dans Muscu
- Onglet Tracking : agenda hebdomadaire pour noter les séances réalisées, préciser leur type et ajouter un détail
- Forum partagé entre appareils via Supabase, prêt à activer avec la configuration décrite ci-dessous
- Réglages d’apparence près du bouton Accueil : cinq thèmes au choix (sombre, clair, océan, forêt et violet), mémorisés sur l’appareil
- Réglages d’apparence près du bouton Accueil : cinq thèmes au choix (sombre, clair, océan, forêt et violet), mémorisés sur l’appareil
- Dans Muscu → Performances, des courbes datées suivent l’estimation du 1RM exercice par exercice au fil des séances, avec le record, la variation depuis le début et le volume d’entraînement
- L’accueil affiche calories et macros, sans bannière de fenêtre alimentaire
- Calendrier et bilan de progression
- Menu de navigation en bas de l’écran sur téléphone ; Accueil reste accessible par son bouton dédié près du compte et n’apparaît pas dans le menu
- Données personnelles sauvegardées sur le téléphone ou l’ordinateur de chaque personne
- Installation sur l’écran d’accueil et accès hors ligne
- Exportation et restauration d’une sauvegarde JSON depuis le profil

## Connecter le forum Supabase

Le forum est prévu pour partager les messages entre les appareils via Supabase. Pour l’activer :

1. Crée un projet Supabase et ouvre son **SQL Editor**.
2. Exécute le script SQL ci-dessous pour créer la table et ses règles d’accès.
3. Dans les réglages API du projet, récupère son URL et sa clé publique **anon/publishable**.
4. Dans `index.html`, remplace les deux chaînes vides des constantes `FORUM_SUPABASE_URL` et `FORUM_SUPABASE_PUBLIC_KEY` par ces valeurs. N’utilise jamais la clé `service_role` ou une clé secrète dans l’application publiée.
5. Publie de nouveau l’application sur GitHub Pages ; tous les utilisateurs de cette version utiliseront le même forum.

```sql
create table if not exists public.forum_messages (
  id bigint generated always as identity primary key,
  author text not null check (char_length(trim(author)) between 1 and 40),
  content text not null check (char_length(trim(content)) between 1 and 500),
  created_at timestamptz not null default now()
);

alter table public.forum_messages enable row level security;
grant select on public.forum_messages to anon, authenticated;
revoke insert on public.forum_messages from anon, authenticated;
grant insert (author, content) on public.forum_messages to anon, authenticated;

drop policy if exists "Anyone can read forum messages" on public.forum_messages;
create policy "Anyone can read forum messages"
  on public.forum_messages for select to anon, authenticated
  using (true);

drop policy if exists "Anyone can post forum messages" on public.forum_messages;
create policy "Anyone can post forum messages"
  on public.forum_messages for insert to anon, authenticated
  with check (
    char_length(trim(author)) between 1 and 40
    and char_length(trim(content)) between 1 and 500
  );
```

**Important :** l’accès sans compte choisi pour ce forum signifie que les pseudos ne sont pas vérifiés et que tout visiteur peut lire ou publier des messages. Les règles limitent la longueur, mais ne fournissent ni modération ni protection anti-spam avancée. Ne publie aucune information personnelle ou sensible. Pour une communauté publique à grande échelle, ajoute une authentification et une modération côté serveur avant l’ouverture.

## Publier sur GitHub Pages

1. Crée un dépôt GitHub et ajoute-y `index.html`, `manifest.webmanifest`, `app-icon.svg`, `sw.js` et `README.md` (le fichier HTML original peut être conservé comme copie).
2. Dans le dépôt, ouvre **Settings → Pages**.
3. Sous **Build and deployment**, choisis **Deploy from a branch**.
4. Sélectionne la branche `main` et le dossier `/ (root)`, puis clique sur **Save**.
5. Attends la fin du déploiement. GitHub affichera l’adresse du site dans **Settings → Pages**.

Le nom du dépôt détermine généralement l’adresse : un dépôt nommé `utilisateur.github.io` utilise `https://utilisateur.github.io/`; un autre nom de dépôt donne une adresse du type `https://utilisateur.github.io/nom-du-depot/`.

## Installer sur un téléphone

Publie d’abord l’application et ouvre son adresse GitHub Pages sur le téléphone.

- **Android (Chrome)** : menu ⋮ → **Installer l’application** ou **Ajouter à l’écran d’accueil**.
- **iPhone (Safari)** : bouton **Partager** → **Sur l’écran d’accueil**.

Une fois installée et ouverte une première fois avec Internet, l’application peut s’ouvrir hors ligne. Garde l’habitude de l’ouvrir dans le même navigateur ou depuis la même icône sur l’écran d’accueil : selon le téléphone, le navigateur et l’application installée peuvent avoir des stockages séparés.

Avant de changer de téléphone ou de navigateur, télécharge une sauvegarde depuis **Profil → Réglages avancés et données**. Sur l’autre téléphone, ouvre SèchePro, choisis **Restaurer une sauvegarde sur ce téléphone** et sélectionne le fichier JSON. Cela transfère les données sans compte ni synchronisation en ligne.

## Données et confidentialité

Chaque personne garde ses données dans le stockage local de son propre téléphone et navigateur. Ces données ne sont pas envoyées à GitHub et ne se synchronisent pas automatiquement entre appareils. Dans **Profil → Réglages avancés et données**, l’application demande au navigateur de protéger le stockage contre le nettoyage automatique lorsqu’il le permet. Cette protection ne peut pas empêcher la suppression manuelle des données, la désinstallation ou la perte du téléphone. Télécharge régulièrement une sauvegarde depuis le profil pour pouvoir récupérer ou transférer tes données. La navigation privée ne conserve généralement pas ces données durablement.

Les valeurs nutritionnelles et estimations d’activité sont indicatives et ne remplacent pas un avis médical ou diététique.
