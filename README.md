SecMobileLab6

LAB 6 :: Analyse statique d'un APK avec MobSF dans la VM Mobexler 

Travail a faire : Ce lab permet de découvrir l'analyse statique d'applications Android (APK) à l'aide de MobSF (Mobile Security Framework), un outil open-source d'analyse automatisée de sécurité mobile. L'analyse s'effectue dans un environnement contrôlé (VM Mobexler) et vise à identifier les vulnérabilités potentielles sans exécuter l'application.

Objectif du Lab : Comprendre le processus d'analyse statique d'un APK .


APK utiliser : apk d'une application free et opensource securisee : cromite dernier version
               apk d'une application dedie test non-securise : FireV dernier version 

Calculer et noter le hash SHA-256 de l'APK pour verifie l'integrite des apk:
<img width="1918" height="1195" alt="image" src="https://github.com/user-attachments/assets/105ced6c-195e-49ee-877f-e05a1fc148b5" />
<img width="1918" height="1195" alt="image" src="https://github.com/user-attachments/assets/49340ebf-bad9-461e-98db-25ebfed403d3" />


<img width="1391" height="875" alt="image" src="https://github.com/user-attachments/assets/55ce477e-c716-4cd1-b8b0-33f1e632b85d" />

<img width="1391" height="875" alt="image" src="https://github.com/user-attachments/assets/2d3c8ad1-459f-4a43-9514-0f3bf8cced48" />


Mobsf sur mobexler VM : 

<img width="1918" height="1195" alt="image" src="https://github.com/user-attachments/assets/17bc66e9-4105-4c69-9323-fe7057da24ba" />

<img width="1918" height="1195" alt="image" src="https://github.com/user-attachments/assets/5d190ad3-8d5d-46f3-a60b-62bc052b8df1" />


Resultat d'analyse de cromite dernier version :: 

<img width="1918" height="1195" alt="image" src="https://github.com/user-attachments/assets/37fd0758-087c-491a-87b9-e2eb866e0058" />

<img width="1918" height="1195" alt="image" src="https://github.com/user-attachments/assets/2c70447f-991f-4793-b95b-d42175da4ca0" />


<img width="1918" height="1195" alt="image" src="https://github.com/user-attachments/assets/c30a08c2-f738-4f2c-9132-c01abdd706cd" />

<img width="1908" height="1195" alt="image" src="https://github.com/user-attachments/assets/2152b7d7-ae1b-4e9e-a4af-4e9148f483c4" />

1. Informations de base (App Information)
Package Name : org.cromite.cromite

Version : 145.0.7632.120

Target SDK : 36 (Android 15), ce qui est très récent et implique des restrictions de sécurité strictes.

Min SDK : 29 (Android 10).

Security Score : 49/100. Ce score peut paraître bas, mais pour un navigateur web (basé sur Chromium), il est souvent dû aux nombreuses permissions nécessaires pour les fonctionnalités web.

2. Analyse du Manifeste (Manifest Analysis)
D'après ta capture "Manifest Analysis", on observe :

Network Security Configuration : L'application utilise un fichier de configuration (@xml/network_security_config). C'est un bon point, car cela permet de définir précisément les politiques de trafic (comme l'épinglage de certificats ou l'interdiction du clair) sans toucher au code Java.

Composants Exportés : Tu as un nombre significatif de composants exposés :

12 Activités exportées : Ce qui est normal pour un navigateur (pouvoir ouvrir des liens depuis d'autres apps).

3 Services, 6 Receivers et 2 Providers exportés.

Risque : Il faudra vérifier si ces composants sont protégés par des permissions spécifiques pour éviter qu'une application malveillante sur le téléphone ne puisse les exploiter.

Network Security :: 

<img width="1908" height="1195" alt="image" src="https://github.com/user-attachments/assets/e07b908a-fb46-4075-9a87-b090521b8cf3" />

1. Analyse des Vulnérabilités Réseau (High Severity)
Le rapport MobSF indique trois problèmes critiques dans la configuration de base (Base config) :

Trafic en clair (HTTP) autorisé (Clear text traffic) :

Statut : HIGH.

Détail : L'application est configurée pour permettre le trafic non chiffré vers tous les domaines (scope: *).

Risque : Une attaque de type Man-in-the-Middle (MitM) peut facilement intercepter les données sensibles transitant entre le navigateur et les sites web.

Confiance envers les certificats installés par l'utilisateur :

Statut : HIGH.

Détail : L'application fait confiance aux CA (Autorités de Certification) ajoutées manuellement par l'utilisateur dans les paramètres Android.

Risque : Cela facilite l'interception du trafic HTTPS par un attaquant (ou un chercheur en sécurité) via un proxy comme Burp Suite ou ZAP, car il suffit d'installer un certificat racine malveillant.

Confiance envers les certificats système :

Statut : WARNING. C'est le comportement standard, mais MobSF le signale pour t'inciter à vérifier si l'application devrait plutôt utiliser du Certificate Pinning.



<img width="1908" height="1195" alt="image" src="https://github.com/user-attachments/assets/ad91b94a-c0c2-4e52-a147-512c773495d1" />


1. Vulnérabilités de Sévérité : MOYENNE (Warning)
Générateur de nombres aléatoires non sécurisé (CWE-330) :

Détail : L'application utilise java.util.Random au lieu de SecureRandom.

Risque : Les valeurs générées sont prévisibles. Si c'est utilisé pour des jetons de session ou des identifiants temporaires, un attaquant peut les deviner.

Accès au Stockage Externe (CWE-276) :

Détail : L'application a des droits de lecture/écriture sur le stockage partagé (External Storage).

Risque : Les données stockées ici sont accessibles par n'importe quelle autre application ayant la permission READ_EXTERNAL_STORAGE.

Divulgation d'adresses IP (CWE-200) :

Détail : Des adresses IP sont présentes en dur dans le code (fichiers hj5.java, ouc.java).

Risque : Cela expose l'infrastructure interne ou des endpoints de test, facilitant la reconnaissance pour un attaquant.

2. Observations de Sévérité : BASSE (Info)
Journalisation d'informations sensibles (CWE-532) :

Détail : L'application utilise des fonctions de Log.

Risque : En mode production, des données sensibles (URL, jetons, infos utilisateur) pourraient se retrouver dans le logcat d'Android, accessible via USB (ADB).

3. Point de Sécurité Positif (Secure)
Protection contre le Tapjacking (MSTG-PLATFORM-9) :

Détail : Présence de mécanismes dans m76.java et x90.java pour détecter si l'écran est recouvert par une autre application.

Bénéfice : Empêche un malware de superposer une fenêtre invisible pour voler les clics de l'utilisateur.







