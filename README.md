# Ecommerce_db

This project contains the SQL schema and scripts for an e-commerce database. 

## Contents
- `Ecomerce_db_ndreamms.sql`: SQL file with the database schema and related queries.

## Usage
1. Open the SQL file in your preferred SQL client (e.g., MySQL Workbench, phpMyAdmin, DBeaver).
2. Execute the script to create the database and tables.
3. Modify or extend the schema as needed for your e-commerce application.

## Project Structure
- **Ecomerce_db_ndreamms.sql**: Main SQL script for the database.









Exercice 3 : Diagramme Entité-Association complet
 
Exercice 4 : Passage au schéma relationnel
 


Exercice 5 :Vérification de la normalisation

1FN — Première forme normale
Règle : chaque attribut est atomique (une cellule = une valeur), pas de liste, pas de groupe répétitif, pas de valeurs multiples dans une colonne.
categorie est en 1FN car chaque attribut contient une seule valeur scalaire. id_parent est un UUID unique ou NULL — pas une liste d'identifiants.
produit est en 1FN car nom, prix_base, statut sont des valeurs atomiques. On aurait pu violer la 1FN en stockant plusieurs catégories dans une colonne (ex. "cat1,cat2") — ce n'est pas le cas, id_categorie est un UUID unique.
article est en 1FN car SKU, prix, actif sont atomiques. Le risque de violation aurait été de stocker plusieurs variantes dans une même ligne — notre architecture décompose chaque variante en un article distinct.
attribut_produit est en 1FN car nom, type, obligatoire sont des valeurs scalaires uniques par ligne.
valeur_attribut est en 1FN car valeur contient une seule chaîne atomique par ligne. C'est précisément le rôle de cette table dans notre architecture EAV : éviter de stocker plusieurs valeurs d'attributs dans une seule cellule.
promotion est en 1FN car libelle, type, valeur, date_debut, date_fin sont tous atomiques. Une violation aurait consisté à stocker plusieurs produits ciblés dans une colonne — id_produit est un UUID unique ou NULL.
stock est en 1FN car qte_disponible, qte_reservee, seuil_alerte sont des entiers atomiques. localisation est une chaîne unique.
mouvement_stock est en 1FN car type, quantite, motif, date_mouvement sont atomiques. Un seul mouvement par ligne, pas de liste.
compte est en 1FN car email, mot_de_passe_hash, statut, role sont des valeurs scalaires uniques. On aurait pu violer la 1FN en stockant plusieurs rôles dans une colonne (ex. "admin,support") — role est ici une valeur unique avec contrainte CHECK.
client est en 1FN car prenom, nom, telephone, date_naissance sont atomiques. Une seule valeur par cellule.
adresse est en 1FN car ligne1, ligne2, code_postal, ville, pays sont des champs atomiques distincts. La décomposition en colonnes séparées est précisément ce qui garantit la 1FN — une seule adresse complète n'est pas stockée dans une colonne unique type "3 rue de Paris, 75001 Paris".
fidelite est en 1FN car points, niveau, date_adhesion sont des valeurs scalaires uniques par client.
mouvement_fidelite est en 1FN car type, points, date sont atomiques. Un seul mouvement par ligne.
code_promo est en 1FN car code, type, valeur, date_expiration, nb_utilisations_max, nb_utilisations_actuel sont des valeurs scalaires. Une violation aurait consisté à stocker plusieurs codes dans une ligne — chaque code promo est une ligne distincte.
panier est en 1FN car statut, date_creation, date_expiration sont atomiques. Les articles du panier ne sont pas stockés dans une colonne de panier — ils sont dans la table ligne_panier, ce qui respecte strictement la 1FN.
ligne_panier est en 1FN car quantite et prix_au_moment sont des valeurs scalaires. Un seul article par ligne.
commande est en 1FN car tous ses attributs sont atomiques. Les deux adresses sont deux FK distinctes (id_adresse_livraison, id_adresse_facturation) et non une liste dans une seule colonne. Les articles commandés ne sont pas stockés ici mais dans ligne_commande.
ligne_commande est en 1FN car quantite, prix_unitaire, remise_ligne sont des valeurs scalaires. Un seul article par ligne de commande.
historique_statut est en 1FN car ancien_statut, nouveau_statut, date_transition sont atomiques. L'historique n'est pas stocké comme une liste dans la table commande — chaque transition est une ligne distincte dans cette table dédiée.
paiement est en 1FN car montant, methode, statut, date_paiement, reference_externe sont atomiques.
tentative_paiement est en 1FN car statut, code_erreur, message_erreur, date_tentative, ip_client sont des valeurs scalaires. Une seule tentative par ligne.
transporteur est en 1FN car nom, code, url_tracking, actif, delai_moyen_jours, contact_email sont atomiques.
expedition est en 1FN car numero_suivi, statut, date_expedition, date_livraison_estimee, date_livraison_reelle sont des valeurs scalaires distinctes. Les trois dates sont trois colonnes séparées — pas une liste.
retour est en 1FN car motif, statut, date_demande, date_reception, montant_rembourse sont atomiques. Les articles retournés ne sont pas stockés ici mais dans ligne_retour.
ligne_retour est en 1FN car quantite, etat, remis_en_stock sont des valeurs scalaires. Un seul article par ligne de retour.

2FN — Deuxième forme normale
Règle : tout attribut non-clé dépend de toute la clé primaire. Ne s'applique que lorsque la clé primaire est composite — une clé simple rend la 2FN automatiquement satisfaite.

Toutes les 25 tables du schéma ShopBase utilisent une clé primaire simple de type UUID. Il n'existe donc aucune clé composite dans le schéma, ce qui signifie que la 2FN est automatiquement et totalement satisfaite pour l'ensemble des tables.
La seule table qui aurait pu présenter un risque est valeur_attribut, dont la sémantique est naturellement composite (un attribut pour un article donné). Nous aurions pu choisir une clé primaire composite (id_attribut, id_article), auquel cas il faudrait vérifier que valeur et ordre dépendent bien du couple entier et non d'un seul des deux. Cette vérification aurait donné : valeur dépend du couple (id_attribut, id_article) — pas de dépendance partielle. La 2FN aurait quand même été respectée. Notre choix d'un UUID id_valeur comme clé primaire simple rend cette question sans objet.

Conclusion générale
Le schéma ShopBase respecte la 1FN sur l'ensemble des 25 tables grâce à trois décisions d'architecture prises en amont : la décomposition des articles du panier et de la commande dans des tables de lignes dédiées, la décomposition de l'adresse en colonnes atomiques distinctes, et l'architecture EAV pour les attributs dynamiques des produits. La 2FN est satisfaite par construction grâce au choix systématique de clés primaires simples UUID. La 3FN est satisfaite sur 24 tables, avec une seule violation corrigée sur ligne_commande (sous_total).
Vérification de la normalisation — ShopBase

categorie est en 3FN car tous ses attributs (nom, slug, niveau, ordre) dépendent directement et uniquement de id_categorie. id_parent est une auto-référence nullable, atomique, sans transitif.
produit est en 3FN car nom, prix_base, statut, date_creation dépendent tous directement de id_produit. id_categorie est une clé étrangère structurelle, pas un attribut dérivé.
article est en 3FN car SKU, prix, poids, actif dépendent directement de id_article. id_produit est une FK légitime. Aucune valeur n'est calculable depuis une autre.
attribut_produit est en 3FN car nom, type, obligatoire dépendent directement de id_attribut. id_categorie est une FK de définition, pas un transitif.
valeur_attribut est en 3FN car la valeur dépend directement de id_valeur. id_attribut et id_article sont deux FK indépendantes, pas de dépendance entre elles.
promotion est en 3FN car libelle, type, valeur, date_debut, date_fin dépendent directement de id_promotion. id_produit nullable est une FK optionnelle, pas un dérivé.
stock est en 3FN car qte_disponible, qte_reservee, seuil_alerte, localisation, date_maj dépendent tous directement de id_stock. Aucun n'est calculable depuis un autre.
mouvement_stock est en 3FN car c'est une table d'audit append-only. type, quantite, motif, date_mouvement dépendent directement de id_mouvement. Aucune dépendance transitive.
compte est en 3FN car email, mot_de_passe_hash, statut, date_creation, role dépendent directement de id_compte. email est UQ mais pas PK — cela ne crée pas de transitif.
client est en 3FN car prenom, nom, telephone, date_naissance, date_suppression_rgpd dépendent directement de id_client. id_compte est une FK de séparation RGPD, pas un attribut dérivé.
adresse est en 3FN car ligne1, code_postal, ville, pays, est_defaut dépendent directement de id_adresse. On note que ville et pays sont des attributs indépendants — on ne dérive pas le pays depuis la ville, donc pas de transitif.
fidelite est en 3FN car points, niveau, date_adhesion dépendent directement de id_fidelite. On note que niveau pourrait théoriquement être dérivé de points (bronze < 100 pts, argent < 500 pts…) — cependant niveau est une décision métier explicite, pas un calcul automatique, ce qui justifie de le stocker indépendamment.
mouvement_fidelite est en 3FN car type, points, date dépendent directement de id_mvt_fidelite. id_fidelite et id_commande sont deux FK indépendantes sans transitif entre elles.
code_promo est en 3FN car code, type, valeur, date_expiration, nb_utilisations_max, nb_utilisations_actuel dépendent directement de id_code_promo. nb_utilisations_actuel est mis à jour transactionnellement — ce n'est pas un dérivé calculable, c'est un compteur géré applicativement.
panier est en 3FN car statut, date_creation, date_expiration dépendent directement de id_panier. id_compte est une FK, pas un attribut transitif.
ligne_panier est en 3FN car quantite et prix_au_moment dépendent directement de id_ligne_panier. prix_au_moment est un snapshot contractuel — il ne dépend pas de article.prix mais de la valeur figée au moment de l'ajout au panier.
commande est en 3FN car statut, date_commande, montant_total, montant_remise dépendent directement de id_commande. montant_remise pourrait sembler transitif via id_code_promo, mais c'est un snapshot contractuel : la remise est figée à la validation de la commande indépendamment de toute modification ultérieure du code promo.
ligne_commande viole la 3FN car l'attribut sous_total dépend de prix_unitaire, remise_ligne et quantite — tous trois étant des attributs non-clés de la même table. La chaîne est : id_ligne → prix_unitaire, remise_ligne, quantite → sous_total. C'est une dépendance transitive caractérisée.
Correction proposée : supprimer sous_total comme colonne stockée et le remplacer par une colonne calculée automatiquement par le moteur de base de données, garantissant la cohérence sans stocker une valeur dérivée.
historique_statut est en 3FN car ancien_statut, nouveau_statut, date_transition dépendent directement de id_historique. Les deux colonnes de statut sont des snapshots d'état indépendants, pas des dérivés l'un de l'autre.
paiement est en 3FN car montant, methode, statut, date_paiement, reference_externe dépendent directement de id_paiement. Aucun n'est calculable depuis un autre.
tentative_paiement est en 3FN car statut, code_erreur, message_erreur, date_tentative, ip_client dépendent directement de id_tentative. id_paiement est une FK, pas un transitif.
transporteur est en 3FN car nom, code, url_tracking, actif, delai_moyen_jours, contact_email dépendent directement de id_transporteur. Aucune dépendance entre attributs non-clés.
expedition est en 3FN car numero_suivi, statut, date_expedition, date_livraison_estimee, date_livraison_reelle dépendent directement de id_expedition. Les trois dates sont des données métier indépendantes, pas des dérivées.
retour est en 3FN car motif, statut, date_demande, date_reception, montant_rembourse dépendent directement de id_retour. montant_rembourse est une décision métier figée, pas un calcul automatique depuis les lignes de retour.
ligne_retour est en 3FN car quantite, etat, remis_en_stock dépendent directement de id_ligne_retour. id_retour et id_article sont deux FK indépendantes sans transitif.

Schéma final corrigé
Une seule table nécessite une modification. Avant correction :
ligne_commande(id_ligne PK, id_commande FK, id_article FK,
               quantite, prix_unitaire, remise_ligne, sous_total)
Après correction — sous_total devient une colonne calculée, plus une valeur stockée indépendamment :
ligne_commande(id_ligne PK, id_commande FK, id_article FK,
               quantite, prix_unitaire, remise_ligne,
               sous_total CALCULÉ = (prix_unitaire - remise_ligne) × quantite)
Les 24 autres tables sont inchangées. Le schéma ShopBase est en 3FN.
























Partie 2
Requêtes SQL
Exercice 6 : Génération des données de test
Nous avons créé un fichier .SQL avec le nombre de valeurs demandé.
 
 

Requêtes R1 à R17 : requête SQL opérationnelle + notation de complexité (O(?)) justifiée
Exercice 7 : Tableau de bord des 17 temps d'exécution + plans EXPLAIN ANALYZE des 3 requêtes les plus lentes

Req	Complexité théorique	Scan principal	Observation	Runtime
R1	O(n log n)	Seq Scan × 3 + Sort	Hash Join sur article/produit/stock, dominé par le Sort final sur p.nom. Seq Scan justifié par le faible volume — basculerait en Index Scan au-delà de ~10 000 lignes	
R2	O(n log n)	O(k)	Tri sur les k lignes retenues	164 msec.
R3	O (n_s + k log k)	O(n_s + n_a + k log k)	① Lire tout stock      → coûte n_s
② Trier les k résultats → coûte k log k
─────────────────────────────────────
Total                  → n_s + k log k	150 msec.
R4	O (n log n)	O (n log n)	La requête ne filtre aucune ligne ,tous les n clients remontent, donc le tri porte sur l'intégralité des données. Or tout algorithme de tri par comparaison a une borne inférieure théorique de O(n log n)  impossible de faire mieux quand on trie n éléments sans index ordonné. Le scan et le join sont linéaires O(n), mais c'est le tri final qui domine et fixe la complexité globale.	170 msec.
R5	O (n log n)	Sort O(n log n)	Aucun filtre ne réduit les lignes , toutes les n_cmd commandes remontent et doivent être triées, ce qui impose incompressiblement O(n_cmd log n_cmd).	159 msec.
R6	O (nlog n)	O (nlog n)	Correspondance 100% parfaite avec l'analyse théorique.	140 msec.
R7	O(n_cmd log n_cmd)	O(n_cmd log n_cmd)	Correspondance 100% parfaite avec l'analyse théorique.	228 msec.
R8	O(n_e)	O(n_e)	Correspondance 100% parfaite avec l'analyse théorique.	179 msec.
R9	O(n_e)	O(n_e)	Correspondance 100% parfaite avec l'analyse théorique.	
R10	O (n log n)	O (n log n)	le plan réel valide intégralement l'analyse théorique. Le nœud Limit apparaît explicitement, ce qui confirme qu'il s'applique après le Sort . PostgreSQL trie tous les k groupes avant de retenir les 10 premiers, exactement comme expliqué. La complexité O(n_cmd) est confirmée, dominée par le scan et l'agrégation sur commande.	162 msec.
R11	O(n_lc)	O(n_lc)	e scan de ligne_commande (70 021 lignes) domine absolument tout le reste. Les optimisations Merge Join et Incremental Sort sont des adaptations PostgreSQL à la très faible cardinalité de cat_parent, sans impact sur la complexité asymptotique globale.	368 msec.
R12	O(n_cmd + n_p + n_e) 	O(n_cmd + n_p + n_e) 	Le filter pushdown de PostgreSQL est particulièrement efficace ici : en filtrant 8 305 lignes avant le join client, il économise autant de lookups dans la hash table des 5 200 clients.	174 msec.
 R13	O(n_lr)	O(n_lr)	le Memoize n'en change pas la borne asymptotique mais réduit significativement le coût constant des lookups sur produit. C'est une optimisation particulièrement efficace ici car ligne_retour contient peu de produits distincts, maximisant le taux de cache hit.	160 msec.
R14	O(n_lc²)	O(n_lc²)	haque table doublée confirme que PostgreSQL traite bien ligne_commande comme deux flux indépendants croisés l'un avec l'autre. La recommandation de matérialisation périodique est pleinement justifiée par ce plan.	450 msec.
R15	O(n_t log n_t)	O(n_t log n_t)	Correspondance parfaite avec l'analyse théorique. C'est le plan le plus propre de la série des requêtes avancées  avec 2 scans, 1 join, 1 agrégat, 1 tri	163 msec.
R16	O(n_cmd)	O(n_cmd)		159 msec.
R17	O(n log n)	O(n log n)		725 msec


Les 3 plus lentes sont : R17, R14  et R 11.

## Author
- Ndreamms

---
Feel free to update this README with more details about your database structure, usage instructions, or project goals.
