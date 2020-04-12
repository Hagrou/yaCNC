# Projet CNC

# Controller sur un raspberry

L'idée est de pouvoir controller en temps réel notre CNC. Pour cela, nous allons utiliser un raspberry surlequel nous installons 

## Deport d'écran

Pour éviter de rester dans le garage pendant que la CNC travail, on va installer le serveur vncserver sur le raspi afin de pouvoir travailler en déport d'écran !

Sur le raspi

    $ sudo apt install vnc4server
    $ vncpasswd 
    Password:
    Verify:
    Would you like to enter a view-only password (y/n)? y
    Password:
    Verify:

    $ vncserver -localhost no 

Sur la machine de controles

    $ remmina


2 parties :

$ sudo apt install java-common


- Controle des moteurs via arduino  => GRBL (exécution de gcode)

- Execution de jcode et controle de l'arduibno bCNC

## Controle Arduino (GRBL)

L'ensemble des mouvements est réalisé par de l'interprétation de
langage, le gcode. Nous allons utiliser le soft GRBL pour cela.

### Installation de l'environnement de compilation

Il faut tout d'abord mettre en place l'environnement de [compilation](http://maxembedded.com/2015/06/setting-up-avr-gcc-toolchain-on-linux-and-mac-os-x/)
pour pouvoir configurer GRBL.

	$ sudo apt update
	$ sudo apt upgrade

Installation de gcc-avr

	$ sudo apt install gcc-avr binutils-avr avr-libc gdb-avr

### Compilation de GRBL

Tout est prêt pour compiler GRBL. Il faut tout d'abord le récupérer
sur [github](https://github.com/grbl/grbl).

	$ mkdir grbl
	$ unzip grbl-master.zip
	$ cd grbl-master
	$ make clean; make all
	


### Configuration de GRBL

Pour cela, j'ai suivi un [tuto](https://www.cours-gratuit.com/cours-arduino/tutoriel-arduino-et-grbl-avec-cnc-shield-v3-pdf) plutot sympa.

La personnalisation se fait à partir du fichier grbl/default.h qui
fait un include sur grbl/default/default_<CNC>.h
#### Fichier default.h
Par default, il pointe sur le fichier grbl/default/default_generic.h

On va donc : 

- dériver grbl/default/default_generic.h en grbl/default/default_yacnc.h

	  $ cp grbl/grbl-master/grbl/defaults/defaults_generic.h grbl/grbl-master/grbl/defaults/defaults_yacnc.h 

- faire point le default.h dessus. Editer le fichier drbl/default.h et changer ajouter :

	  #ifdef DEFAULTS\_YACNC
		#include "default/defaults_yacnc.h"
	  #endif


#### Configuration de default\_yacnc.h

Le fichier default\_yacnc.h (Yet Another CNC) va contenir l'ensemble
des caractéristiques de notre CNC. Pour cela il faut :

https://lebearcnc.com/configurer-et-parametrer-grbl/


Les premiers paramètres sont liés aux caractéristiques des moteurs pas à pas ainsi que les pas de vis:

      #define MOTOR_STP 200 // number of step per turn (Angle de pas : 1.8 degree)  
      #define MICRO_STP 16  // Micro stepping 1/16 step ? TODO: valide 1?              
	  #define SCREW_PITCH_MM 2 // Pas de 2mm pour les vis sans fin                 
	  

Ce qui permet de déduire le nombre de tour pour avancer d'1 mm :

	  // Grbl generic default settings. Should work across different machines.            
	  #define DEFAULT_X_STEPS_PER_MM (MICRO_STP*MOTOR_STP/SCREW_PITCH_MM)
      #define DEFAULT_Y_STEPS_PER_MM (MICRO_STP*MOTOR_STP/SCREW_PITCH_MM)
      #define DEFAULT_Z_STEPS_PER_MM (MICRO_STP*MOTOR_STP/SCREW_PITCH_MM)

On définit ensuite la course max de chaque axe (mm) : 

	  #define DEFAULT_X_MAX_TRAVEL 240.0 // mm // (130) TODO 
	  #define DEFAULT_Y_MAX_TRAVEL 480.0 // mm // (131) TODO                               
	  #define DEFAULT_Z_MAX_TRAVEL 75.0 // mm // (132) TODO    
      
Les vitesses max : 
	
	  #define DEFAULT_X_MAX_RATE 380.0 // (110) mm/min
	  #define DEFAULT_Y_MAX_RATE 380.0 // (111) mm/min   
      #define DEFAULT_Z_MAX_RATE 220.0 // (112) mm/min  
      
Enfin on active les detecteurs de fin de course

	  #define DEFAULT_HOMING_ENABLE 1  // activation des detecteurs de fin de course
      #define  DEFAULT_HOMING_DIR_MASK 0 // move positive dir
	   #define DEFAULT_HOMING_FEED_RATE 25.0 // (24) Vitesse lente/précise (mm/min)
       #define DEFAULT_HOMING_SEEK_RATE 380.0 // (25) Vitesse de déplacement jusqu'à déclencher les capteurs mm/min     
	   #define DEFAULT_HOMING_DEBOUNCE_DELAY 250 // (26) msec (0-65k) Tps de filtrage de rebond du capteur              
	  #define DEFAULT_HOMING_PULLOFF 1.0 // (27) mm decalage des fins des capteurs                                     

#### Configuration de config.h

Pour activer la configuration yacnc, il faut :

- remplacer la ligne 
	 
	  #define DEFAULTS_GENERIC 
	 
  par 
  
      #define DEFAULTS_YACNC



## Compilation 

- Trouver le port usb de connexion avec l'arduino

ls /dev/tty*

=> on chercher quelque chose comme /dev/ttyACM?

lancer la compilation

	$ make
	
	
Flasher l'arduino

	$ make flash
	
    
    https://github.com/gnea/grbl/wiki/Grbl-v1.1-Configuration#20---soft-limits-boolean

 python -m serial.tools.miniterm /dev/ttyACM0 115200


    List of available ports:
$ python -m serial.tools.list_ports

Terminal:
$ python -m serial.tools.miniterm <port_name>

$ python -m serial.tools.miniterm /dev/ttyUSB0
