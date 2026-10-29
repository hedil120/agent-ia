# TP 3 — De « Hello agent » à la boucle agentique

## Étape 0 — Préparer l'environnement

### Question 0.1
La clé API est stockée dans le fichier `.env` pour éviter de l'exposer directement dans le code. Le fichier `.env` figure dans `.gitignore` afin que Git ne l'ajoute pas au dépôt et ne publie pas la clé sur GitHub.

## Étape 1 — Un appel, c'est une requête HTTP

### Question 1.1
Dans la réponse JSON de l'API Chat Completions, le texte généré se trouve dans `choices[0].message.content`.

### Question 1.2
La requête doit contenir :
- l'adresse de l'API ;
- la clé API, envoyée dans l'en-tête HTTP `Authorization` ;
- le modèle et les messages, envoyés dans le corps JSON.

L'en-tête `Content-Type` indique également que le corps est au format JSON.

### Question 1.3
Le SDK construit et envoie la requête HTTP, puis transforme la réponse en objets Python faciles à utiliser. Il ne garantit pas que la réponse soit correcte et ne donne pas automatiquement une mémoire au modèle.

## Étape 2 — Disséquer la réponse

### Observation
Avec le lancement par défaut, le résultat indique `finish_reason: length`, 84 tokens en entrée et 400 tokens en sortie. La réponse a atteint la limite de sortie configurée.

### Question 2.1
### Question 2.1
Avec la limite par défaut de 400 tokens, le modèle a utilisé 400 tokens en sortie et `finish_reason` vaut `length`. Avec la limite de 20 tokens, il a utilisé 20 tokens en sortie et `finish_reason` vaut également `length`. Dans les deux cas, la génération s'arrête parce qu'elle atteint la limite de tokens.

### Question 2.3
Avec la limite de 20 tokens, le texte visible est vide alors que 18 tokens de raisonnement ont été utilisés. Le modèle a consommé une grande partie de la limite pour son raisonnement avant de produire une réponse visible.

### Exercice 2 — Calcul du coût
La fonction `cout(usage, prix_entree, prix_sortie)` doit calculer :

`(prompt_tokens × prix_entree + completion_tokens × prix_sortie) / 1 000 000`

Avec 84 tokens en entrée, 400 tokens en sortie, un prix de 0,10 $ par million de tokens en entrée et de 0,40 $ par million en sortie, un appel coûte 0,0001684 $.

Pour 200 appels par étudiant et 100 étudiants, cela représente 20 000 appels, soit un coût total fictif de **3,368 $**.
### Question 2.2
Le nombre de tokens en entrée comprend non seulement la question, mais aussi la structure des messages et les éventuelles instructions ajoutées par le modèle ou le protocole.

## Étape 3 — Le modèle est probabiliste

### Observation
À la température 0, les cinq réponses étaient `TunisGo`. À la température 1.2, les réponses étaient `TunisSprint`, `TunisExpress`, `SaharaSprint` et `SaharaZip`, avec `SaharaSprint` répété.

### Question 3.1
À la température 0, j'ai obtenu un seul nom distinct. À la température 1.2, j'ai obtenu quatre noms distincts.

### Question 3.2
Même à température basse, une sortie n'est pas toujours garantie identique. Un programme doit donc vérifier et traiter les réponses du modèle au lieu de supposer qu'elles seront toujours exactement au même format.

### Question 3.3
Pour produire du JSON valide, je choisirais une température basse pour favoriser des réponses plus stables. Pour générer des idées, je choisirais une température plus élevée pour favoriser la diversité.
## Étape 4 — L’expérience de l’oubli

### Question 4.1
Non, le modèle ne connaît pas mon prénom au second appel, car chaque appel est indépendant. Le contexte du premier appel n’est pas automatiquement transmis au deuxième.

### Question 4.2
ChatGPT semble se souvenir des messages précédents parce que l’application conserve l’historique de la conversation et le transmet au modèle à chaque nouvel appel. Le modèle peut ainsi utiliser le contexte précédent pour répondre.

## Étape 5 — Construire la mémoire d’un chatbot

### Question 5.1
Au troisième tour, le modèle comprend « sa » grâce aux messages précédents transmis dans l’historique. Il peut relier ce pronom à Tunis, la capitale de la Tunisie.

### Question 5.2
Le nombre de tokens en entrée augmente au fil de la conversation : 84 au premier tour, 162 au deuxième, 190 au troisième et 291 au quatrième. Cela s’explique par le fait que le programme transmet progressivement davantage d’historique au modèle.

### Question 5.3 — Historique
La commande `/historique` affiche les messages de l’utilisateur et les réponses de l’assistant conservés dans la conversation. Elle permet de vérifier que l’historique sert de mémoire au chatbot.
### Question 6.2 — Date d’aujourd’hui
Le modèle répond : « Aujourd’hui, c’est le 29 octobre 2026. » Le nombre de tokens en entrée est de 87.

### Question 6.3 — Date limite des connaissances
Le modèle indique que ses connaissances s’étendent jusqu’en juin 2024. Le nombre de tokens en entrée est de 88.
## Étape 7 — Le prompt système

### Question 7.1
Oui, Sami répond différemment du modèle à l’étape 6. Il se présente comme l’assistant virtuel du service de scolarité de l’école d’informatique à Tunis. Cela s’explique par la présence d’un prompt système qui définit son identité, son rôle et les limites de son domaine d’intervention. Il refuse la recette de couscous, qui ne concerne pas la scolarité, et ne révèle pas une autre identité lorsque l’utilisateur lui demande d’ignorer ses instructions.
## Étape 8 — Ce que le modèle peut faire

### Question 8.1 — Les réponses du modèle sont-elles correctes ?

**Question 2 : Un calcul exact**

Le modèle donne `450618883`, ce qui correspond exactement à la vérité `450618883`. La réponse est donc correcte.

**Question 3 : Le contenu de mon dossier**

Le modèle refuse d’accéder directement au système de fichiers et explique comment afficher les fichiers avec les commandes `dir` ou `ls`. Sa réponse ne correspond pas à la liste exacte attendue. Elle est donc incorrecte pour cette tâche, même si les conseils donnés sont utiles.

### Question 8.2 — Que peut-on conclure ?

Le modèle réussit le calcul exact, mais il ne peut pas afficher directement le contenu du dossier avec ses seules capacités de conversation. Cela montre qu’un modèle de langage peut répondre à certaines questions, mais qu’il ne peut pas automatiquement exécuter des actions sur le système de fichiers.

### Question 8.3 — Qui peut obtenir la bonne réponse ?

Pour afficher le contenu réel du dossier, un programme doit exécuter une commande comme `ls` et transmettre son résultat au modèle. Le modèle seul ne connaît pas automatiquement les fichiers présents sur le système. Un outil lui permettrait d’obtenir cette information et de répondre précisément.
## Étape 9 — Un outil fait à la main

### Question 9.1 — Comment fonctionne l’appel d’outil ?

Le modèle effectue deux appels :
1. Il demande l’action `calculer` avec l’expression `48763 * 9241`.
2. Le harnais exécute le calcul et renvoie le résultat `450618883`.
3. Le modèle effectue un deuxième appel pour donner la réponse finale.

### Question 9.2 — Qui effectue réellement le calcul ?

Le modèle choisit l’opération à effectuer, mais c’est le harnais qui exécute réellement le calcul. Le résultat est ensuite transmis au modèle, qui le renvoie comme réponse finale.

### Conclusion

Un outil permet au modèle de réaliser une tâche qu’il ne peut pas exécuter directement. Le modèle décide quand utiliser l’outil, tandis que le programme exécute l’action.
## Étape 10 — La boucle agentique

### Question 10.1 — Comment fonctionne la boucle agentique ?

L’agent fonctionne en plusieurs étapes :
1. Le modèle analyse la tâche et choisit une action.
2. Le programme exécute l’action demandée.
3. Le résultat de l’action est renvoyé au modèle sous forme d’observation.
4. Le modèle utilise cette observation pour choisir l’action suivante ou donner sa réponse finale.

### Question 10.2 — Quel est le rôle des outils ?

Les outils permettent à l’agent d’obtenir des informations ou d’effectuer des calculs. Dans cet exemple, l’outil `heure_actuelle` donne la date et l’heure, puis le modèle calcule le nombre de minutes restantes avant minuit.

### Résultat obtenu

À 21 h 07, l’agent a calculé qu’il restait 173 minutes avant minuit. Il a utilisé deux tours : le premier pour obtenir l’heure et le deuxième pour fournir la réponse finale.

### Conclusion

La boucle agentique suit le principe « Penser → Agir → Observer → Recommencer ». Elle permet au modèle d’enchaîner plusieurs actions jusqu’à obtenir une réponse.
## Étape 11 — Les outils natifs

### Question 11.1 — Comment fonctionne l’appel d’outil natif ?

Le modèle effectue trois tours :
1. Il demande l’outil `heure_actuelle` pour obtenir la date et l’heure.
2. Il demande l’outil `calculer` avec l’expression `(24 - 21) * 60 - 8`, qui donne 172 minutes.
3. Il fournit sa réponse finale à l’utilisateur.

### Question 11.2 — Quelle est la différence avec l’étape 10 ?

À l’étape 10, le modèle choisit une action en produisant un objet JSON dans son texte, puis le programme interprète cette réponse. À l’étape 11, l’API fournit directement les appels d’outils dans le champ `tool_calls`, avec le nom de l’outil et ses arguments.

### Conclusion

Les outils natifs structurent les appels d’outils dans la réponse de l’API. Le programme
## Étape 12 — Le contexte initial

### Question 12.1 — Pourquoi fournir un contexte initial ?

Le contexte initial définit l’identité de l’assistant, ses règles, l’environnement et les informations sur l’utilisateur. Il permet au modèle de répondre de manière plus adaptée et de connaître les informations disponibles dès le début de la conversation.

### Question 12.2 — Quel est l’intérêt de la section environnement ?

Elle fournit des informations comme la date, l’heure, le système d’exploitation et le dossier de travail. Ces informations permettent à l’assistant de répondre à des questions qui nécessitent le contexte actuel, au lieu de devoir le deviner.

### Question 12.3 — Quel est le rôle des outils ?

Les outils permettent à l’agent d’obtenir des informations ou d’effectuer des calculs. Ici, l’agent utilise l’outil `calculer` pour déterminer le nombre de minutes restantes avant minuit.

### Question 12.4 — Quelle est la différence entre la taille estimée et les tokens réellement comptés ?

La taille estimée est calculée approximativement à partir du nombre de caractères, en utilisant la règle d’un token pour quatre caractères. Le fournisseur compte réellement les tokens selon sa propre méthode de tokenisation. Dans cet essai, l’estimation est de 302 tokens, tandis que le fournisseur en compte 361.

### Conclusion

Un contexte initial bien structuré aide l’agent à connaître son rôle, les règles à respecter, l’environnement dans lequel il travaille et les informations concernant l’utilisateur. Les outils lui permettent ensuite d’effectuer des actions précises et de fournir une réponse adaptée.
## Étape 12 — Construire le contexte initial

### Question 12.1 — Partie A : comparez les deux réponses et les tokens en entrée.

Sans la section Environnement, le modèle indique qu'il ne connaît pas la date actuelle. Avec cette section, il répond que dans 10 jours, nous serons un dimanche. Le contexte Environnement fournit la date du jour, ce qui permet au modèle de répondre.

Sans environnement : 144 tokens. Avec environnement : 221 tokens. La section Environnement ajoute donc 77 tokens à l'entrée.

### Question 12.2 — Comparez l'estimation aux tokens réellement comptés. Pourquoi l'écart ?

L'estimation est d'environ 302 tokens, tandis que le fournisseur compte réellement 361 tokens au premier appel. L'écart est de 59 tokens. La règle d'un token pour quatre caractères est approximative : la tokenisation dépend du texte et du modèle. Le fournisseur compte également les éléments de structure du contexte.

### Question 12.3 — L'agent vous a-t-il appelé par votre prénom ? D'où le tient-il ?

Oui, l'agent m'a appelée Yasmine. Il connaît mon prénom parce qu'il est indiqué dans la section Utilisateur du contexte initial. Cette information lui est transmise dans le prompt système.

### Question 12.4 — Pourquoi la date doit-elle être calculée à chaque démarrage ?

La date doit être calculée à chaque démarrage avec `datetime.now()` pour rester à jour. Si elle était écrite une seule fois dans le prompt, elle deviendrait incorrecte avec le temps.

### Question 12.5 — Proposez une information de contexte inutile.

L'adresse personnelle de l'utilisateur serait inutile pour cet agent si aucune tâche ne nécessite cette information. Il vaut mieux ne transmettre que les données utiles afin d'éviter d'alourdir le contexte et de partager des informations personnelles sans nécessité.

### Exercice 12 — Mesurer le coût de chaque brique

Le contexte initial est composé de plusieurs briques : identité, règles, environnement, utilisateur et outils. Pour mesurer leur coût, il faut envoyer le prompt après l'ajout de chaque brique, relever `prompt_tokens`, puis calculer la différence avec la mesure précédente. Les valeurs doivent être relevées lors des essais.

### Exercice 12 — Mesurer le coût de chaque brique

J’ai mesuré le nombre de tokens en entrée après l’ajout progressif de chaque brique du contexte initial.

| Brique du contexte | Tokens totaux | Tokens ajoutés |
|---|---:|---:|
| Identité | 106 | 106 |
| Règles | 154 | 48 |
| Environnement | 220 | 66 |
| Utilisateur | 249 | 29 |
| Outils | 351 | 102 |

Le contexte complet avec les outils représente **351 tokens** selon les mesures du fournisseur. Les règles ajoutent 48 tokens, l’environnement 66 tokens, les informations utilisateur 29 tokens et les outils 102 tokens.

Cette mesure montre que chaque information transmise au modèle a un coût. Il est donc préférable de conserver les informations et les outils utiles à la tâche pour éviter d'augmenter inutilement la consommation de tokens.