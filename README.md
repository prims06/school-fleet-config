# Fedora School — dépôt de configurations Fleet signées

La configuration propre à chaque établissement se trouve dans :

- **Colombe (tenant de test)** : [`clients/sbfoirln4rms22uj/config.json`](clients/sbfoirln4rms22uj/config.json)
- **Démo** : [`clients/wsew5xl3ocm6wbvq/config.json`](clients/wsew5xl3ocm6wbvq/config.json)

Chaque fichier `config.json` a son `config.json.sig` (Ed25519). **Ne pas changer directement un JSON publié sans le faire re-signer puis republier par le mécanisme Fedora Console / agent local** : l'app refuserait une signature périmée. La clé privée de signature ne se met jamais dans ce dépôt. Aucun fichier live ni signature n'a été modifié lors de cette documentation.

## Où configurer les modèles de bulletins

FEDORA School a **deux mécanismes différents**. Ils sont facultatifs et se trouvent tous les deux à la RACINE du `config.json` signé :

### A. Adapter le bulletin FEDORA existant (recommandé)

Ajouter `school.bulletinTemplates`, avec la clé de l'ordre et uniquement les champs souhaités. Cela conserve les calculs et le moteur PDF intégrés.

```json
{
  "school": {
    "bulletinTemplates": {
      "PRI_FR": {
        "accentColor": "#0E7490",
        "header": { "headTitle": "Directeur" }
      },
      "SEC_FR": {
        "visualPreset": "minesec-apc-a4-v1",
        "header": { "headTitle": "Principal" }
      },
      "SEC_EN": {
        "visualPreset": "minesec-apc-a4-v1"
      }
    }
  }
}
```

**Attention : exemple de fragment, pas config complète.** Conserver `school.orders`, `school.sections` et les blocs existants. Les clés de `school.bulletinTemplates` acceptent aussi un cycle ou niveau connu, avec priorité **niveau > cycle > ordre > défaut**. Les 8 ordres sont `MAT_FR`, `MAT_EN`, `PRI_FR`, `PRI_EN`, `SEC_FR`, `SEC_EN`, `TEC_FR`, `TEC_EN`. Les options visuelles sont limitées à celles de `src/domain/school/bulletinDefaults.js` de l'app. Les libellés officiels, la conformité ministérielle et les calculs se valident séparément.

Un fragment modifiable est fourni dans [`examples/bulletin-overrides.fragment.json`](examples/bulletin-overrides.fragment.json).

### B. Remplacer complètement le dessin par du HTML Handlebars

Ajouter `documentTemplates` à la RACINE (et **non** sous `school`) :

```json
{
  "documentTemplates": {
    "bulletin": {
      "template": "<style>/* A4, CSS local */</style><article>{{student.name}}</article>",
      "size": "A4"
    },
    "preschoolBulletin": {
      "template": "<article>{{student.name}}</article>",
      "size": "A4"
    }
  }
}
```

**Exemple de syntaxe uniquement, non utilisable tel quel comme bulletin réglementaire.** `documentTemplates.bulletin` est un gabarit HTML scolaire numérique ; `documentTemplates.preschoolBulletin` doit rester qualitatif et traiter `domains` / `competencies`. `documentTemplates.receipt` concerne le reçu de frais. Une clé **absente** ou un template vide laisse le rendu Fedora natif. Ne pas remplacer un modèle officiel par l'exemple simplifié.

Le champ `template` reçoit **le contenu HTML/Handlebars entier en chaîne** (pas le chemin du fichier, ni une URL). Le formulaire « Documents » de Fedora Console, depuis l'évolution correspondante, permet de saisir les templates pour un **nouvel enrôlement School** ; les clients existants exigent une mise à jour Fleet signée. Les ressources externes ne sont pas disponibles hors ligne : images/polices locales ou intégrées seulement.

**Modèle source prêt à adapter** : [`school-management-app/docs/authoring/examples/bulletin-a4.hbs`](https://github.com/prims06/school-management-app/blob/erp/docs/authoring/examples/bulletin-a4.hbs) et [guide du format des données](https://github.com/prims06/school-management-app/blob/erp/docs/authoring/document-templates.md). L'exemple complet doit traiter **un élève ET un lot**, ainsi que les périodes séquence/trimestre/annuel ; un template compilable peut sinon produire une page fausse sans lever d'erreur.

## Procédure de publication / recette obligatoire

1. Depuis Fedora Console ou l'agent de publication disposant de la clé **en dehors** de GitHub, préparer la modification du seul tenant visé (tester d'abord sur Colombe). **Fusionner** le fragment avec le JSON existant ; ne remplacer ni `supabase.enc`, ni `tenant.localDbName`, ni `auth`, ni `license`, ni les offres de cycles.
2. Vérifier le JSON, le schéma Fleet, la longueur HTML et les ressources offline. Incrémenter `configVersion` à la republication, re-signer `config.json` avec la clé d'éditeur, puis publier **JSON + signature ensemble**. Ne jamais exposer la clé privée. Fedora Console préserve les autres types de templates à la republication partielle.
3. Sur un appareil de test, vérifier que la signature est acceptée et que le nouveau `configVersion` est effectivement reçu. Prévisualiser puis exporter PDF et ZIP sur des élèves MAT/PRI/SEC/TEC, FR/EN, avec et sans connexion et **un élève / classe entière** ; surveiller la pagination, les calculs, les mentions et l'historique.
4. Si une erreur survient, republier une configuration antérieure **re-signée** avec une version supérieure, jamais modifier le fichier signé en place sans protocole.

## Répartition des responsabilités

- **Fleet** : identité, blocs graphiques, templates optionnels et options école.
- **Base locale Dexie et Supabase** : matières/programmes/notes/permissions, jamais des modèles HTML de déploiement au hasard.
- **Fedora Console** : construction, publication, signature et provisioning des tenants. Le gabarit SQL Console doit être à jour du SQL canonique School (sans quoi un nouveau tenant ne part pas sur le bon schéma).

Ce dépôt ne contient que des configurations publiées et de la documentation ; **aucune migration BD ni clé privée**.
