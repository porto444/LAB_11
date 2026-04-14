# Rapport TP : Sécurité Android & Instrumentation avec Frida 

Ce dépôt contient les travaux réalisés pour l'exercice de contournement des protections de Root sur Android en utilisant l'outil d'instrumentation dynamique **Frida**.

**Étudiant :** Ahmed  
**Cible :** RootBeer Sample (Détection de Root)

---

##  Résumé du projet

L'objectif était de manipuler le comportement d'une application Android en temps réel pour masquer la présence du "Root" sur un émulateur. Nous avons injecté des scripts JavaScript dans le processus de l'application pour modifier les valeurs de retour des fonctions de sécurité.

---

##  Configuration de l'environnement

- **Frida CLI :** v17.9.1
- **Frida-Server :** v17.9.1 (Déployé sur l'appareil)
- **Appareil :** Émulateur Android Studio (Architecture `x86_64`)
- **ADB :** Android Debug Bridge activé
<img width="854" height="316" alt="Capture d&#39;écran 2026-04-14 153331" src="https://github.com/user-attachments/assets/c5af53e0-c118-4bc3-8e0e-dfcc9e80709d" />

---

##  Étapes de réalisation

### 1. Préparation du serveur
Installation et lancement du binaire `frida-server` sur l'émulateur :
```bash
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
```

<img width="732" height="64" alt="Capture d&#39;écran 2026-04-14 153413" src="https://github.com/user-attachments/assets/a34fd46f-e56a-4935-98fd-bba1a3c3419b" />
<img width="1007" height="104" alt="Capture d&#39;écran 2026-04-14 153525" src="https://github.com/user-attachments/assets/bdc2b17b-0233-40a3-b5cb-8c6153d7bc09" />
<img width="998" height="625" alt="Capture d&#39;écran 2026-04-14 153605" src="https://github.com/user-attachments/assets/3dcee520-1368-41fe-b227-70e6c267a45a" />

---

## 2. Vérification de la connectivité
Vérification que l'appareil est bien détecté et que Frida peut lister les processus :
frida-ps -Uai

---

## 3. Injection du script de Bypass (Java)
Utilisation du script bypass_root.js pour hooker la couche Java :

Modification de Build.TAGS -> release-keys.
Forçage de isRooted() -> false.
Blocage de la détection des fichiers su et busybox.
frida -U -f com.scottyab.rootbeer.sample -l bypass_root.js --no-pause

---
## 4. Traçage Natif
Utilisation de frida-trace pour analyser les appels système vers la couche C/C++ (NDK) :
frida-trace -U -i open -i access com.scottyab.rootbeer.sample

---
## Résultats obtenus
Test	     Avant Frida        	Après Frida
Statut    Root	🔴 Rooted (Détecté)	🟢 Not Rooted (Contourné)
Logs     Console	-	[+] Hook Build.TAGS successful
