# PATH TRAVERSAL

## What is a 'Path traversal'?

This vulnerability allows reading arbitrary files on the server filesystem by manipulating the page parameter with ../ sequences ("remonte d'un niveau de dossier"), named 'Path Traversal' technic.
C'est a dire, profiter d'un chemin de fichier afficher pour sortir du dossier prévu par l'application et accéder à n'importe quel fichier ailleurs sur le serveur.

Par exemple, si une application construit un chemin:
<pre>
/var/www/html/pages/
</pre>
et qu'on le modifie en ajoutant un fichier de base a tester
<pre>
/var/www/html/pages/../../../../etc/passwd
</pre>

On sort du dossier pages/ pour atteindre un fichier complètement différent, ailleurs sur le disque.C'est une faille de securite!

Local File Inclusion (LFI)
C'est une conséquence du Path Traversal, au lieu de juste lire le contenu d'un fichier, l'application va l'inclure et l'exécuter comme du code.

Path Traversal seul → lire le contenu d'un fichier
LFI → lire + executer 


## How to find the vulnerability?

To find the flag of this vulnerability you need to try one of the basics test for hacking.
<pre>
/etc/passwd
</pre>

'/etc/passwd' -> ce fichier est le test universel de cette faille car il existe par défaut sur tout système Linux, il est toujours range au meme endroit et il est lisible par n'importe quel utilisateur du système.
*More infos about basics hacks test in the general README*

On the main page, you need to test by adding the /etc/passwd with a suit of '../' sequences until you go back up out of the folder.

<pre>
http://10.11.249.50/index.php
</pre>
to
<pre>
http://10.11.249.50/index.php?page=../../../../../../../../etc/passwd
</pre>

Le principe : le serveur n'a pas vérifié rigoureusement le nombre de niveaux ou n'a pas neutralisé correctement la séquence ../, ce qui t'a permis de sortir du dossier prévu par l'application pour accéder à des fichiers du système.

Avant d'atteinte le bon nombre de "remontees", vous allez tomber sur un pop up qui vous affichera "WTF?" ou "Almost", c'est que vous etes sur la bonne voie! Ces pop up sont declenches car le developper a surement bloque l'acces excepte pour le bon nombre de "remontees" jusqu'au fichier racine, ou alors par ce que le fichier demande n'existe pas a cette adresse.


## Suggestions to protect from Path traversal

### Why it was a vulnerability?
Dans le fichier '/etc/passwd', il y a tous les noms d'utilisateurs du système qui peuvent par exemple servir a tenter un brute-force de connexion SSH.
Il y a aussi les shells utilisés, qui révèlent souvent quels comptes sont de vrais comptes humains vs des comptes système.
Ce serait mieux qu'ils ne tombent pas entre de mauvaises mains!

### How to do protect better?
le paramètre ?page= est utilisé directement pour choisir quel fichier afficher, sans validation stricte de sa valeur.
However, not every file path will necessarily be readable — this can depend on file permissions, additional filtering logic, or the exact file location, which would require further testing to map precisely.

1. Liste blanche stricte (whitelist) — la solution la plus fiable
Au lieu de essayer de bloquer des motifs dangereux (../, ce qui est toujours contournable comme tu l'as vu), on définit à l'avance la liste exacte des pages autorisées, et on refuse tout le reste

C'est la différence fondamentale entre blacklist (bloquer ce qu'on sait être dangereux — toujours incomplet) et whitelist (n'autoriser que ce qu'on sait être sûr — bien plus robuste). C'est un principe de sécurité général, pas juste pour cette faille.
2. Utiliser basename()
Cette fonction PHP ne garde que le nom du fichier final d'un chemin, en supprimant tout ce qui précède (les dossiers, et donc les ../) :
php$page = basename($_GET['page']);
include($page . '.php');
Avec ça, même si tu envoies ../../../../etc/passwd, basename() ne retient que passwd, donc le chemin redevient passwd.php dans le bon dossier — la tentative de traversée est neutralisée automatiquement, peu importe le nombre de ../ ou la façon de les écrire (encodés, doublés, etc.).

3. Vérifier que le chemin final reste bien dans le dossier autorisé
Une vérification plus technique mais robuste : on calcule le vrai chemin absolu final du fichier demandé, et on vérifie qu'il commence bien par le dossier autorisé.
php$base_dir = realpath('/var/www/html/pages/');
$requested = realpath($base_dir . '/' . $_GET['page'] . '.php');

if ($requested === false || strpos($requested, $base_dir) !== 0) {
    die("Accès refusé");
}
include($requested);
realpath() résout tous les ../ et donne le vrai chemin final sur le disque — si ce chemin final ne commence pas par le dossier prévu, on bloque. C'est plus complexe à écrire mais très fiable.

4. Ne jamais faire confiance à un simple str_replace() ou détection de motif
C'est exactement ce qui causait le "Almost" dans Darkly — bloquer juste la chaîne ../ littérale est toujours contournable : encodage URL, doublement de caractères, chemins absolus, etc. Un filtre par blacklist donne une illusion de sécurité, jamais une vraie protection.

5. Limiter les permissions au niveau système (défense en profondeur)
Indépendamment du code PHP, configurer le serveur web pour qu'il n'ait pas les droits de lecture sur des fichiers sensibles en dehors de son propre dossier (open_basedir en PHP, qui restreint nativement les fonctions de fichiers à un dossier précis) :
ini; dans php.ini
open_basedir = /var/www/html/
Même si une faille de code existe, le serveur PHP lui-même refusera d'aller chercher des fichiers hors de ce périmètre.


The most reliable protection is to use a strict whitelist of allowed page names instead of trying to filter out dangerous patterns like ../, which can always be bypassed through encoding or repetition. Functions like basename() or realpath() combined with a path verification add further protection by neutralizing directory traversal attempts regardless of how they are written. Server-level restrictions such as PHP's open_basedir directive provide an additional layer of defense even if the application code itself contains a flaw.

### Conclusion

This echoes the OWASP category A01:2021 – Broken Access Control, which deals with the fact that access controls are not being properly enforced on the server side.une application ne restreint pas correctement l'accès à des ressources qui devraient être protégées.
