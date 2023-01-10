# TP3---Traitement-d-un-signal-ECG

<a name="retour"></a>
## Sommaire :
1. [ Objectifs. ](#objectif)
2. [ Introduction.](#intro)
3. [ Suppression du bruit provoqué par les mouvements du corps. ](#part1)
4. [ Suppression des interférences des lignes électriques 50Hz. ](#part2)
5. [ Amélioration du rapport signal sur bruit. ](#part3)
6. [ Identification de la fréquence cardiaque avec la fonction d’autocorrélation. ](#part4)


<a name="objectif"></a>
### **1. Objectifs:**

 Suppression du bruit autour du signal produit par un électrocardiographe
 Recherche de la fréquence cardiaque.

#### Commentaires :
 Il est à remarquer que ce TP traite en principe des signaux continus.Or, l'utilisation de Matlab suppose l'échantillonnage du signal. Il faudra donc être vigilant par rapport aux différences de traitement entre le temps continu et le temps discret.

#### Tracé des figures :
  Toutes les figures devront être tracées avec les axes et les légendes des axes appropriés
  
#### Travail demandé :
  Un script Matlab commenté contenant le travail réalisé et des commentaires sur ce que vous avez compris et pas compris, ou sur ce qui vous a semblé intéressant ou pas, bref tout commentaire pertinent sur le TP.
  
  <a name="intro"></a>
### **2. Introduction:**

Un électrocardiogramme (ECG) est une représentation graphique de l’activation électrique du cœur à l’aide d’un électrocardiographe. Cette activité est recueillie sur un patient allongé, au repos, par des électrodes posées à la surface de la peau.L’analyse du signal ECG est utile dans le but de diagnostiquer des anomalies cardiaques telles qu’une arythmie, un risque d’infarctus, de maladie cardiaque cardiovasculaire ou encore extracardiaque.


Ci-dessous, voici un schéma représentant une représentation classique d’une courbe d’un ECG. Ce schéma se nomme un « Complexe QRS » mettant en évidence le bon fonctionnement d’un cycle cardiaque.


<img width="306" alt="intro" src="https://user-images.githubusercontent.com/93081417/211406384-48c972c1-2f1d-46e2-805c-bc11bdbfe8b3.png">


L’onde P représente la première étape du cycle où les oreillettes (ou atriums) se contractent permettant le passage du sang, à travers les valves auriculoventriculaires, vers les ventricules.


Ensuite, le complexe QRS symbolise à la fois la contraction ventriculaire (permettant l’éjection du sang vers les artères) notamment par le pic en R, dans le même temps,le relâchement des oreillettes entraîne le remplissage de celles-ci en attente d’un nouveau cycle).

L’onde T représente le relâchement des ventricules suite à leur contraction.L’enchaînement de ces complexes permet par ailleurs de déterminer la fréquence cardiaque, c’est-à-dire le nombre de cycle cardiaque par unité de temps. Une fréquence cardiaque normale est comprise entre 60 et 100 battements par minute, 
en dessous de cette valeur, le patient est en « bradycardie », au-dessus de cette valeur,le patient est en « tachycardie ».

Les signaux ECG sont contaminés avec différentes sources de bruits. Les bruits de hautes fréquences sont provoqués par l’activité musculaire extracardiaque et les interférences dues aux appareils électriques, et des bruits de basses fréquences provoqués par les mouvements du corps liés à la respiration, les changements physicochimiques induits par l’électrode posée sur la peau et les micro variations du flux sanguin. Le filtrage de ces bruits est une étape très importante pour faire un diagnostic réussi.


$~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~$ [ (Revenir au sommaire) ](#retour)
***


<a name="part1"></a>
### **3. Suppression du bruit provoqué par les mouvements du corps:**


 ####  **1- Sauvegarder le signal ECG sur votre répertoire de travail, puis charger-le dans Matlab à l’aide la commande load.**
 
 ```matlab
 load("ecg.mat");
 x=ecg;
 ```
  ####  **2- Ce signal a été échantillonné avec une fréquence de 500Hz. Tracer-le en fonction du temps, puis faire un zoom sur une période du signal.**

```matlab
fs=500;
N=length(x)
ts=1/fs
%tracer ECG en fonction de temps
t=(0:N-1)*ts;
plot(t,x);
title("le signal ECG");
```
<img width="788" alt="ecg" src="https://user-images.githubusercontent.com/93081417/211408306-44cfee11-ac10-4331-8871-cd54f67e89b8.png">

####  **3- Pour supprimer les bruits à très basse fréquence dues aux mouvements du corps,on utilisera un filtre idéal passe-haut. Pour ce faire, calculer tout d’abord la TFD du signal ECG, régler les fréquences inférieures à 0.5Hz à zéro, puis effectuer une TFDI pour restituer le signal filtré.**



```matlab
% le spectre Amplitude
 y = fft(x);
 f = (0:N-1)*(fs/N);
 fshift = (-N/2:N/2-1)*(fs/N);
 plot(fshift,fftshift(abs(y)))
title("spectre Amplitude")

%suppression du bruit des movements de corps

h = ones(size(x));
fh = 0.5;
index_h = ceil(fh*N/fs);
h(1:index_h)=0;
h(N-index_h+1:N)=0;

ecg1_freq = h.*y;
ecg1 =ifft(ecg1_freq,"symmetric");
```

####  **4- Tracer le nouveau signal ecg1, et noter les différences par rapport au signal d’origine.**

```matlab
plot(t,ecg1);
```
<img width="820" alt="ecg1" src="https://user-images.githubusercontent.com/93081417/211409922-defee704-ceb9-45f7-b249-7368d9484001.png">

==> La différence par rapport au signal d'origine
```matlab
plot(t,x-ecg1);
```

<img width="809" alt="x" src="https://user-images.githubusercontent.com/93081417/211410333-2f2f4009-5996-4f3b-ba5e-115a7fa709f6.png">


$~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~$ [ (Revenir au sommaire) ](#retour)
***


<a name="part2"></a>
### **4. Suppression des interférences des lignes électriques 50Hz:**

Souvent, l'ECG est contaminé par un bruit du secteur 50 Hz qui doit être supprimé
####  **5. Appliquer un filtre Notch idéal pour supprimer cette composante. Les filtres Notch sont utilisés pour rejeter une seule fréquence d'une bande de fréquence donnée.**

```matlab
Notch = ones(size(x));
fcn = 50;
index_hcn = ceil(fcn*N/fs)+1;
Notch(index_hcn)=0;
Notch(index_hcn+2)=0;

ecg2_freq = Notch.*fft(ecg1);
ecg2 =ifft(ecg2_freq,"symmetric");
```
####  **6. Visualiser le signal ecg2 après filtrage.**
```matlab
 plot(t,ecg2);
  title("signal filtré")
```
<img width="827" alt="ecg2" src="https://user-images.githubusercontent.com/93081417/211414498-b290acec-933c-4592-bfdc-ed0932cb958d.png">

<a name="part3"></a>
### **5. Amélioration du rapport signal sur bruit:**

Le signal ECG est également atteint par des parasites en provenance de l’activité musculaire extracardiaque du patient. La quantité de bruit est proportionnelle à la largeur de bande du signal ECG. Une bande passante élevée donnera plus de bruit dans les signaux, et limiter la bande passante peut enlever des détails importants du signal.

####  **7. Chercher un compromis sur la fréquence de coupure, qui permettra de préserver la forme du signal ECG et réduire au maximum le bruit. Tester différents choix, puis tracer et commenter les résultats.**

```matlab
pass_bas = zeros(size(x));
fcb = 20;
index_hcb = ceil(fcb*N/fs);
pass_bas(1:index_hcb)=1;
pass_bas(N-index_hcb+1:N)=1;

ecg3_freq = pass_bas.*fft(ecg2);
ecg3 =ifft(ecg3_freq,"symmetric");

 subplot(211)
plot(t,ecg,"linewidth",1.5);

subplot(212)
 plot(t,ecg3);


```
<img width="791" alt="bruit" src="https://user-images.githubusercontent.com/93081417/211416364-193b5074-2256-4c34-92a6-cf39ed01d994.png">

==>On remarque qu'il y a du bruit

on test avec 30 HZ

<img width="803" alt="zwin" src="https://user-images.githubusercontent.com/93081417/211416427-4ca02a69-c573-4296-9e0b-2322b71e271b.png">


==>On remarque que le bruit est à-peu-près éliminé


####  **8. Visualiser une période du nouveau signal filtré ecg3 et identifier autant d'ondes que possible dans ce signal (Voir la partie introduction).**
```matlab
plot(t,ecg3);
xlim([0.5 1.5])
title('ecg3')
```

$~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~$ [ (Revenir au sommaire) ](#retour)
***

<a name="part4"></a>
### **6. Identification de la fréquence cardiaque avec la fonction d’autocorrélation:**
La fréquence cardiaque peut être identifiée à partir de la fonction d'autocorrélation du signal ECG. Cela se fait en cherchant le premier maximum local après le maximum global (à tau = 0) de cette fonction.


####  **9. Ecrire un programme permettant de calculer l’autocorrélation du signal ECG, puis de chercher cette fréquence cardiaque de façon automatique. Utiliser ce programme sur le signal traité ecg3 ou ecg2 et sur le signal ECG non traité.**
NB : il faut limiter l’intervalle de recherche à la plage possible de la fréquence cardiaque.

```matlab
[c,lags] = xcorr(ecg3,ecg3);
 stem(lags/fs,c)
```
####  **10. Votre programme trouve-t-il le bon pouls ? Commenter.**

==>oui  le programme a trouvé le bon pouls
<img width="800" alt="last" src="https://user-images.githubusercontent.com/93081417/211542971-d7fce668-d447-45f1-98ca-b861b68d42d8.png">


###### Frequence = 0.914*60 = 54.8400 Hz


$~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~$ [ (Revenir au sommaire) ](#retour)
***

## Réalisé par : SAHEL Bouchra
## Filiére : Robotique et Cobotique .
## Encadré par : Pr. Ammour Alae .

