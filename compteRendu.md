# PSTL

## Semaine1 (04/fev/2020)
- tester OMicroB (Linux et Mac) chemin complet : installation + exécution
- écriture d’un exemple simple pour Arduino (à confirmer) et/ou Micro:bit + exécution sur le matériel
  test de cet exemple sur PC et sur le simulateur actuel
- tester les environnements Arduino (et Micro:bit) en C classiques pour voir les fonctionalités

## Semaine2 (11/fev/2020)
- #### deja fait :
  - installation réussie de la branche `microbit-without-mbed` sur macosx

  - programme basic tourne sur le matériel

- #### choses à faire cette semaine :
  - tester le simulateur sur la branche ‘master’ pour Arduino

  - adapter le simulateur dans la branche `microbit-without-mbed` pour faire tourner des programmes ocaml

  - définir les entrées/sorties de base : écran 5x5 et 2 boutons poussoirs dans le simulateur pour micro:bit
  - voir comment rendre générique le simulateur : que faut-il mettre dans target pour pouvoir ajouter une nouvelle architecture (ce qui permettra d’éviter les open Avr en dur)

## Semaine3 (18/fev/2020)

- #### petit cr de réunion :
  - réussir  de tester le simulateur  de pin,circuit, arduino sur la branch master

  - essayé de faire un simulateur de "circuit " pour microbit

  - comprendre faire d'abord des simulateur spécifique pour mcirobit et puis on verra les trucs en commun pour faire la version générique après

- #### choses à faire cette semaine :

  - comprendre le code source du simulateur (définition des pin, d'écran)

  - faire un demo simulateur avec simple entrée sortir pour microbit

  - faire un naïve simulateur pin pour microbit

## Semaine4 (25/fev/2020)
- #### Questions
  - Questions: On a définit les pins et ports de micro:bit, mais on n'a pas encore compris comment les fonctionner... Comment exécuter un programme en utilisant ces pins définit ?

  - Question2: Pour notre simulateur, sa tâche est que nous montrer le résultat de micrô-contrôlleur sur PC.
Donc il faut envoyer le programme compilé (fichier .hex ou .elf) au simulateur, ensuite tourner ce programme sur simulateur. Mais pour l'instant on n'arrive pas à compris comment interprèter .elf.

- #### Réalisé
  - la définition des pins et ports

## Semaine5 (03/Mars/2020)
- Comprendre le processus de compilation.
  1. `test.ml` est compilé par `ocamlc` et produit `test.byte`.
  ```
  CAMLLIB=/usr/local/lib/omicrob ocamlc -g -w A -safe-string -strict-sequence -strict-formats -ccopt -D__OCAML__ -custom -verbose -ccopt -DDEBUG=2 -I /usr/local/lib/omicrob/targets/avr /usr/local/lib/omicrob/targets/avr/avr.cma -I /usr/local/lib/omicrob/targets/avr/arduboy /usr/local/lib/omicrob/targets/avr/arduboy/arduboyPins.cmo -open Avr -open ArduboyPins test.ml -o test.byte
  ```
  2. Commande exécuté produit un erreur \
  `gcc: error: /tmp/camlprimca2b44.c: No such file or directory`\
  puisqu'il n'y a pas de fichier '/tmp/camlprimca2b44.c', donc je saute cette commande.
  ```
  gcc -O2 -fno-strict-aliasing -fwrapv -Wall -fno-tree-vrp -D_FILE_OFFSET_BITS=64 -D_REENTRANT -DCAML_NAME_SPACE  -Wl,-E -o 'test.byte'   '-L/usr/local/lib/omicrob/targets/avr' '-L/usr/local/lib/omicrob/targets/avr/arduboy' '-L/usr/local/lib/omicrob' -D__OCAML__ -DDEBUG=2 '/tmp/camlprimca2b44.c' '-lcamlrun' -I'/usr/local/lib/omicrob' -lm  -ldl -lpthread
  ```
  3. `test.byte` est traité par `ocamlclean`
  ```
  ocamlclean test.byte -o test.byte
  ```
  4. `bc2c` compile `test.byte` à `test.c`
  ```
  bc2c -stack-size 2048 -heap-size 256 -gc MAC -arch 64 test.byte -o test.c
  ```
  5. `g++` compile `test.c` à `test.elf`, `-DDEBUG=1,2,3,4` permet de choisir le mode de debug.
  ```
  g++ -D __PC__ -g -Wall -O -std=c++11 -DDEBUG=2 -I /usr/local/include/omicrob/simul test.c -o test.elf
  ```
  6. `avr-g++` compile `test.c` à `test.avr`
  ```
  avr-g++ -g -fno-exceptions -Wall -std=c++11 -O2 -Wnarrowing -Wl,-Os -fdata-sections -ffunction-sections -Wl,-gc-sections -mmcu=atmega32u4 -DF_CPU=16000000 -DDEBUG=2 -I /usr/local/include/omicrob/avr -I /usr/local/include/omicrob/avr/arduboy test.c -o test.avr
  ```
  7. `avr-objcopy` compile `test.avr` à `test.hex` qui est le fichier exécutable de microcontrolleur
  ```
  avr-objcopy -O ihex -R .eeprom test.avr test.hex
  ```

## Semaine6 (01/Avril/2020)
  On a compris que la simulateur est resemble à une application web. Il sépare en 2 partie, la partie du client et serveur.
  - Partie client est programmée par ocaml dans `/src/simulator/`, il nous montre un UI de simulateur permette de afficher les résultats de simulation et l'interaction par les bouttons.

  - Partie serveur est codée par C dans `/src/byterun/`, il permet d'interprèter le fichier exécutable `.elf`, par exemple `test.elf`, l'ensemble des instructions et functions sont définites dans le fichier `test.c`. La fonction `main` se trouve dans `/src/byterun/vm/interp.c`, on peut trouver les actions de chaque instruction, c'est un peut similaire avec le projet de MINI-ZAM.

  Donc, on va commencer de tavailler à partir d'ici, interprèter les instrctions et affichier le résultat sur l'interface du simulateur en utilisant multiprocessus. Pour l'instant, le plupart de tests sont passée correctement, certaine tests ne fonctionnent pas bien comme `queens`, `recfun`, `recval`, etc. Ensuite je vais trouver une solution pour envoyer le résultat du côté serveur au client. Finalement, réaliser l'interaction entre client et serveur.
  
