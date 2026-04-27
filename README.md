# Rapport TP : Sécurité Android & Instrumentation avec Frida 

Ce dépôt contient les travaux réalisés pour l'exercice de contournement des protections de Root sur Android en utilisant l'outil d'instrumentation dynamique **Frida**.

**Étudiant :** Ahmed  
**Cible :** UnCrackable Level 1 (OWASP MSTG)

---

##  Résumé du projet

L'objectif était de manipuler le comportement d'une application Android sécurisée en temps réel pour masquer la présence du "Root". Contrairement à une application classique, UnCrackable 1 se ferme immédiatement s'il détecte un root ou un debugger. Nous avons dû neutraliser les mécanismes de fermeture forcée et les vérifications de fichiers sensibles.

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
frida-ps -Uai | findstr uncrackable

---

## 3. Injection du script de Bypass (Java)
Utilisation du script bypass_root.js adapté pour UnCrackable 1. Ce script réalise trois actions critiques :
Hook de System.exit : Empêche l'application de se fermer après avoir détecté le root.
Hook de File.exists : Simule l'absence des binaires su et busybox.
Logs temps réel : Affiche chaque tentative de détection bloquée dans la console.
<img width="1309" height="687" alt="Capture d&#39;écran 2026-04-22 162321" src="https://github.com/user-attachments/assets/a58f169c-697a-4621-94a2-7de9f6dc8d99" />


---
## 4. Traçage Natif
Utilisation de frida-trace pour identifier les appels système plus profonds (couche NDK) :

````Bash
frida-trace -U -f owasp.mstg.uncrackable1 -i "open*" -i "stat*"
````
<img width="1463" height="576" alt="Capture d&#39;écran 2026-04-22 162744" src="https://github.com/user-attachments/assets/cf7a9f12-59ad-482f-a620-f59997d35029" />
<img width="744" height="477" alt="Capture d&#39;écran 2026-04-22 162753" src="https://github.com/user-attachments/assets/ed10893d-0a65-45ca-8b88-2a5880339cef" />


Observation : Nous avons identifié que l'application utilise intensivement stat64() pour scanner le système de fichiers. Un second script (bypass_native.js) a été utilisé pour retourner -1 lors de l'accès aux chemins suspects identifiés dans le trace
---
## Résultats obtenus
<img width="1167" height="681" alt="Capture d&#39;écran 2026-04-22 164747" src="https://github.com/user-attachments/assets/7977f3a7-1f05-4849-bd4c-6b904d1f7dcd" />

Conclusion : L'application UnCrackable 1 a été instrumentée avec succès. Toutes les couches de détection (Java et Native) ont été identifiées et contournées, permettant ainsi d'accéder à l'interface principale de l'application malgré l'état "Rooté" de l'émulateur.
