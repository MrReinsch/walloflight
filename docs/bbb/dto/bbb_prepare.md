#Vorbereitung des BeagleBone Black - Device Tree,PRU & Co
Damit die Module PRU0 und PRU1 des AM3359 auch die P8 und P9 Header des
BeagleBone Black ansprechen können, müssen einige Änderungen vorgenommen werden. Seit der Linux Kernel Version 3.8 können Hardwaremodule auch zur Laufzeit aus dem Userspace nachgeladen. Anfänglich war dies nur im Kernalspace oder per DeviceTree ([http://www.devicetree.org/Main_Page](http://www.devicetree.org/Main_Page "DeviceTree.org")) möglich. Statt den DeviceTree umzuschreiben, diesen anschließend neuzukompilieren und das System neuzustarten, werden
DeviceTreeOverlays benutzt. In unserem Anwendungsfall, wäre eine Änderung
des DeviceTree auch möglich, da sich zur Laufzeit die Hardanforderungen nicht ändern. Um in Zukunft kompatible zu sein, ist das Nutzen des Overlays sicher die
bessere Alternative. Das Overlay wird ebenfall kompiliert, aber es überschreibt
den DeviceTree nur zur Laufzeit, so dass dieser nicht geändert wird.
Ich möchte an dieser Stelle darauf hinweisen, dass ich hier erstmal nur die Pinmuxing  DeviceTreeOverlays für die Pru0 und die Pru1 zugänglichen Pins auf den Headern P8 und P9 beschreibe.

**Zunächst sind eine Informationen notwendig, dazu werden folgende Dokumente benötigt:**

1. BeagleBone Black System Reference Manual (SRM) [Rev. A6A]
2. BeagleBone Schematic (SCM) [REV 6B]
3. AM335x Arm Cortex A8 Technical Reference Manual (TRM)

**Welche Pins auf den Header P8 und P9 können durch die Pru0 bzw. Pru1 angesprochen werden?**  
Im SRM in der Sektion **6.12** PRU-ICSS auf der Seite 79 steht in Table 11 alles was notwendig ist. 
PRU0 kann **8** Ausgänge und PRU1 **13** direkt ansprechen. **Folgende Ausgänge können beschrieben werden:**  
  
    PRU0: 	GPIO1_13,GPIO1_12,GPIO3_21,GPIO3_19,SPI1_D0,SPI1_D1,SPI1_CS0,SPI1_SCLK   
    PRU1: 	GPIO2_6,GPIO2_7,GPIO2_8,GPIO2_9,GPIO2_10,GPIO2_11,GPIO2_12,GPIO2_13  
    		GPIO2_22,GPIO2_23,GPIO2_24,GPIO1_30,GPIO1_31  
    
**Welche Hardware ist zusätzlich an diese Pins angeschlossen?**  
Dazu im SCM die Pins bzw. Netznamen abgleichen.  
Ich habe die Netznamen aus dem SRM an das SCM angepasst:  
  
    PRU0_r30_14 GPIO1_12 	- 	P8_12  
    PRU0_r30_15 GPIO1_13 	- 	P8_11  
    PRU0_r30_7 GPIO3_19 	- 	P9_27  
    PRU0_r30_5 GPIO3_21 	- 	P9_25 [Y4 OSC HDMICLK / disable HDMI ]  
    PRU0_r30_1 SPI1_D0 		- 	P9_29,U11-P24 [GPIO3_15]  
    PRU0_r30_2 SPI1_D1 		- 	P9_30 [GPIO3-16]  
    PRU0_r30_3 SPI1_CS0 	- 	P9_28,U11-P25 [GPIO3_17]  
    PRU0_r30_0 SPI1_SCLK 	- 	P9_31,U11-P23 [GPIO3_14]  
    PRU1_r30_0 LCD_DATA0 	- 	P8_45,U11-P15 [GPIO2_6] C47p0F,PU100K,PD100K  
    PRU1_r30_1 LCD_DATA1 	- 	P8_46,U11-P13 [GPIO2_7] C47p0F,PU100K,PD100K  
    PRU1_r30_2 LCD_DATA2 	- 	P8_43,U11-P12 [GPIO2_8] C47p0F,PU100K,PD100K + Taster S2 NO [BOOT]  
    PRU1_r30_3 LCD_DATA3 	- 	P8_44,U11-P11 [GPIO2_9] C47p0F,PU100K,PD100K  
    PRU1_r30_4 LCD_DATA4 	- 	P8_41,U11-P10 [GPIO2_10] C47p0F,PU100K,PD100K  
    PRU1_r30_5 LCD_DATA5 	- 	P8_42,U11-P7 [GPIO2_11] C47p0F,PU100K,PD100K  
    PRU1_r30_6 LCD_DATA6 	- 	P8_39,U11-P6 [GPIO2_12] C47p0F,PU100K,PD100K  
    PRU1_r30_7 LCD_DATA7 	- 	P8_40,U11-P3 [GPIO2_13] C47p0F,PU100K,PD100K  
    PRU1_r30_8 LCD_VSYNC 	- 	P8_27,U11-P21 [GPIO2_22] R0Ohm  
    PRU1_r30_9 LCD_HSYNC 	- 	P8_29,U11-P22 [GPIO2_23] R0Ohm  
    PRU1_r30_10 LCD_VSYNC 	-	P8_28,U11-P4 [GPIO2_24] R22Ohm  
    PRU1_r30_12 MMC1_CLK 	- 	P8_21,U13-PM6 [GPIO1_30] PU10K  
    PRU1_r30_13 MMC1_CMD 	- 	P8_20,U13-PM5 [GPIO1_31] PU10K  

**Was heißt das jetzt?**  
HDMI muss disabled werden. GPIO1-30 und GPIO1-31 sollten nicht benutzt werden, weil das Onboard Flashmemory diese Pins benutzt. Im Notfall könnte eine SD-Karte benutzt werden, aber erstmal werden diese Pins nicht benutzt! Alle anderen Beschlatungen sind relativ unkritisch auch der Spannungsteiler an den LCD-Pins, sobald nach dem Start die I/O Konfiguration geladen wurde. Unkritsche Signale (Latch)sollten auf P9-25 und/oder auf GPIO2-8 gelegt werden. 
Es stehen von PRU0 - 8 GPO's und von PRU1 - 11 GPO's zur Verfügung. `Hinweis: '-'='_' Für die Pinbezeichungen `

**Anforderung:**  
Die Kacheln benötigen jeweils ein Reset,Latch,SPI-CLK und SPI-DO. Der Reset kann aus dem Userspace angesprochen werden können. Somit fällt dieser hier raus, da die PRU wichtigeres zu tun hat. Benötigt werden min. 11 SPI Schnittstellen, CLOCK und LATCH werden distributiert. PRU0 und PRU1 werden vorerst asynchron laufen, so dass jeweils pro PRU ein GPO für die Clock genutzt wird. Das Latch werde ich versuchen synchorn zu halten. Ich bin mir z.Z. noch nicht sicher wie. Jede PRU latched aber erstmal für sich selbst. Eine weitere Anforderung ist die Möglichkeit 12 Kacheln in einem Strang zu hängen. Dabei ist die Anzahl der Stränge geringer als bei 11 Kachel pro Strang. Dies hat eigentlich nur Auswirkungen auf ASM-Programm. Im weiteren Verlauf wird immer nur von 12 die Rede sein, es sollte klar sein, dass 11 auch ohne weiteres Möglich ist!

**Daten**  
Jeder PRU-Core hat für Daten ein 8K RAM und   
beide haben noch einmal einen shared RAM von 12K.   
Das macht dann 28K RAM. Ich rechne mal:  
Jede Kachel hat 64 RGB Leds: 192 Byte / Bild  
Jeder Strang hat 12 Kacheln: 2304 Byte / Bild  
Jeder WoL hat 12 Stränge:	 27648 Byte / Bild  
Von der Theorie her passt alles.  

**Ablauf sieht dann wie folgt aus:**   
Aus dem Userspace werden die Nutzdaten in die PRU geladen. Ob das ASM Prgramm jeweils mit Übertragen werden muss oder ob es einmal geladen wird, ist noch zu klären. Die Nutzdaten werden dann im Userspace ggf. vorsortiert, sofern es notwendig ist. Die PRU'S werden nur die Daten übertragen und das Latch bedienen! Jede Pru berechnet dann für 6 Datenwörter jeweils, ob das entsprechende Bit low oder high ist. Das Ergebnis wird dann in einem Register(16Bit-Wort) festgehalten. Im Anschluss daran wird das Register auf den Output(R30) gelegt. Dann wird der Clock Pin high gesetzt. Bei einer Datenrate von ca. 2MBit "nopt" jeder PRU für eine gewisse Zeit(ca. 100 Cycle pro High/Low) abzgl. Zeit für Berechnung und Daten holen. Nach 12 übertragenden Bytes wird das Latch(Kachel ab Rev.3.1) gesendet. Nach 2304 Byte wird ein weites Latch gesendet. Dieses Latch sollte synchron gesendet werden, d.h. PRU1 und PRU0 müssen sich über interne Signal irgendwie synchronisieren. z.B. PRU0 wartet dann auf PRU1 oder umgekehrt.    

**PRU Anforderung**  
Jede PRU stellt 6 SPI Schnittstellen zur Verfügung. Nach dem 12 Byte übertragen wurden, wird ein Latch (high für min. 50us) gesendet. Während dieser Zeit darf **kein** weiteres Byte gesendet werden! Außerdem muss ggf. anschließend etwas gewartet werden, dazu darf dass Latch auch länger high bleiben. Wichtig ist nur, dass das Latch nicht während einer Übertragung high bleibt. Dies führt in der Kachel dazu, dass der Byte Counter resetet wird und somit keine gültigen Daten Übertragen werden. Nach dem Latch/Warten können wieder 12 Bytes übertragen werden. Wurden alle Bytes übertragen(12 * 192 Byte), muss noch eine weiteres Latch übertragen werden. Dann ist das Bild gültig und die Daten werden übernommen bzw. der Bildbuffer in der Kachel wird getauscht. Genaue Zeiten werden noch ermittelt Kachelsoftware ab **Rev.3.1** 

**GPIO PINMUXING**  
Jeder GPIO muss über Register konfiguriert werden. Die meisten GPIO's können u.a. mit alternate functions belegt werden. Normalerweise wird das enstsprechende GPIO Register dazu direkt beschrieben. Das geht hier allerdings nicht so einfach, da der Zugriff aus dem Userspace auf die Register des A9 nicht einfach so zulässig ist. Im Prinzip kann auf eine bereits geschrieben Library für die GPIO's zurück gegriffen werden, sofern diese existiert. Ich möchte hier den Weg via DeviceTreeOverlay gehen. Allerdigs müsser wir zuerst den HDMI abstellen, da sonst einige GPIO's nicht zur Verfügung stehen. 

**Abstellen HDMI cape**  
Eine Anleitung wie das HDMI Cape disabled wird ist auf folgender Seite nachzulesen:  
[http://http://www.logicsupply.com/blog/2013/07/18/disabling-the-beaglebone-black-hdmi-cape/](http://http://www.logicsupply.com/blog/2013/07/18/disabling-the-beaglebone-black-hdmi-cape/ "logicsupply.com")  

Anleitung:  
1. Verbinden z.B. per SSH und als root einloggen  
2. Mounten der FAT-Partion mit: **mount /dev/mmcblk0p1  /mnt/card**  
3. Öffnen der Datei uENv.text mit: **nano /mnt/card/uEnv.txt**  
4. Inhalt der Datei ändern auf: **optargs=quiet capemgr.disable_partno=BB-BONELT-HDMI,BB-BONELT-HDMIN**  
5. Speichern mit: **Ctrl-X, Y**  
6. Unmounten der Partition: **umount /mnt/card**  
7. Neustarten   
Nach dem Neustart kann mit cat /sys/devices/bone_capemgr.*/slots nachgeshen werden, ob das HDMI Cape richtig konfiguriert wurde. Vor dem HDMI Eintrag sollte sich kein 'L' befinden("ff: P-O--"), ggf. sind es zwei Einträge.

Nach dem das HDMI Cape disabled wurde, kann jetzt das DeviceTreeOverlay geschrieben werden.

**DeviceTreeOverlay**  
Der DeviceTree beschreibt wie oben bereits erwähnt die Hardware. Das DeviceTreeOverlay ist kurz gesagt nur ein Aussschnitt dieser Beschreibung der zur Laufzeit nachgeladen werden kann, d.h. der eigentliche DeviceTree wird nicht permanent überschrieben. Es soll hier nicht darum gehen was genau ein DeviceTree bzw. ein Overlay ist. Wer interesse hat, unter :[http://www.devicetree.org/Main_Page](http://www.devicetree.org/Main_Page "DeviceTree.org")  
Ich habe unter: [http://derekmolloy.ie/beaglebone/beaglebone-gpio-programming-on-arm-embedded-linux/](http://derekmolloy.ie/beaglebone/beaglebone-gpio-programming-on-arm-embedded-linux/ "Derek Molloy") eine recht ausführliche Beschreibung für DeviceTreeOverlay etc. gefunden. Derek Molloy beschreibt auf seine Seite und in den Videos viele nutzlich Dinge, die auch für dieses Projekt interressant sind. Nun aber zum DeviceTreeOverlay.
Im DTO steht im Prinzip nur wie das dem Hardwaremodule konfiguriert wird. Ich habe die Strutur des DTO erstmal weggelassen um hier nur das wesentliche zu zeigen. Hier eine Zeile aus dem Beispiel von Derek Molloy:  

0x078 0x07 /* P9_12 60 OUTPUT MODE7 - The LED Output */   

Der erste Wert beschreibt den Offset und der zweite Wert die
Funktion bzw. Konfiguration. Der berechnet sich wie folgt:  
Offset = Address - 44e10800, mit Address aus dem TRM/SRM. Es ist etwas kompliziert die Adressen raus zu suchen. Es ist auch so, dass der Mode für die jeweilige Pruss auch schwer zu finden ist. Ich empfehle
dafür das Pinmux Tool von TI. 
[http://processors.wiki.ti.com/index.php/Pin_Mux_Utility_for_ARM_MPU_Processors](http://processors.wiki.ti.com/index.php/Pin_Mux_Utility_for_ARM_MPU_Processors "Pinmux Tool")  
Zu Entwicklungsbeginn noch nicht vorhanden aber jetz:  
[http://elinux.org/Ti_AM33XX_PRUSSv2](http://elinux.org/Ti_AM33XX_PRUSSv2 "Pru Programming") , siehe Tablle: **Beaglebone PRU connections and modes** 

Ich fasse aber trotzdem einmal kurz zusammen:  
Die Funktion der meisten Pins des Controllers können unterschiedlich belegt werden. Damit die unterschiedlichen Funktionen sich nicht gegenseitig stören, wird die Belegung bzw. das Multiplexen der Funntionalität eines Pins von einem Controller, dem PinMux-Controller geregelt. Genauer gesagt macht das das Conf_Module siehe TRM section 9.3.1.50. Leider geht aus dem TRM nicht genau hervor welche Mode für die jeweilig Pruss verwendet werden muss. Auch hier hilf das PinMux Tool von TI oder ein Blick in das SRM auf Seite für die Pinheader.  
Wichtig für den Mode: Das Output Register der jeweiligen Pru ist R30.
Das Overlay sieht dann wie folgt aus:  
    
    /*
     * Copyright (C) 2012 Texas Instruments Incorporated - http://www.ti.com/
     *
     * This program is free software; you can redistribute it and/or modify
     * it under the terms of the GNU General Public License version 2 as
     * published by the Free Software Foundation.
     */
    /dts-v1/;
    /plugin/;
    
    /{
      
          compatible = "ti,beaglebone-black";
          part-number = "BBB-WoL-CubeXXL_DTO";
          version = "00A0";
         //board-name = "BBB-Wol-CubeXXL-Carrier";
         //manufacturer = "RT-Labor";
     
       exclusive-use =
        /* the pin header uses */
        "P8.27", /* Reset0 GPIO_OUT */
        "P8.29", /* Reset1 GPIO_OUT */
        "P9.25", /* Latch0 Pru0 */
        "P9.31", /* SCKL0  PRU0 */
        "P9.29", /* SDO0_0 PRU0 */
    	"P9.30", /* SDO0_1 PRU0 */
        "P9.28", /* SDO0_2 PRU0 */
        "P9.27", /* SDO0_3 PRU0 */
        "P8.12", /* SDO0_4 PRU0 */
        "P8.11", /* SDO0_5 PRU0 */
        "P8.43", /* LATCH1 PRU1 */
        "P8.45", /* SCKL1  PRU1 */
        "P8.46", /* SDO1_0 PRU1 */
        "P8.44", /* SDO1_1 PRU1 */
        "P8.41", /* SDO1_2 PRU1 */
        "P8.42", /* SDO1_3 PRU1 */
        "P8.39", /* SDO1_4 PRU1 */
        "P8.40", /* SDO1_5 PRU1 */
        "pruss";
    
       fragment@0 {
          target = <&am33xx_pinmux>;
            __overlay__ {
               pinctrl_Wol_Cube_IO: pinmux_BBB {
    			  pinctrl-single,pins = <
                     /*adr mode   register   gpio header Pin function*/ 
                     0x0e0 0x0f /*PRU1-r30-8	GPIO2_22  P8-27 U5  Reset0 GPIO_OUT*/
                     0x0e4 0x0f /*PRU1-r30-9	GPIO2_23  P8-29 R5  Reset1 GPIO_OUT*/
                     0x1ac 0x05 /*PRU0-r30-7    GPIO3_21  P9-25 A14 LATCH0 PRU0 */
                     0x190 0x05 /*PRU1-r30-0	GPIO3_14  P9-31 A13 SCKL0  PRU0 */
                     0x194 0x05 /*PRU0-r30-1	GPIO3_15  P9-29 B13 SDO0_0 PRU0 */
                     0x198 0x05 /*PRU0-r30-2	GPIO3_13  P9-30 D12 SDO0_1 PRU0 */
                     0x19c 0x05 /*PRU0-r30-3	GPIO3_17  P9-28 C12 SDO0_2 PRU0 */
                     0x1a4 0x05 /*PRU0-r30-5	GPIO3_19  P9-27 C13 SDO0_3 PRU0 */
                     0x030 0x06 /*PRU0-r30-14   GPIO1_12 P8-12 T12 SDO0_4 PRU0 */
                     0x034 0x06 /*PRU0-r30-15   GPIO1_13 P8-11	R12 SDO0_5 PRU0 */
                     0x0a8 0x05 /*PRU1-r30-2	GPIO2_8	  P8-43 R3  LATCH1 PRU1 */
                     0x0a0 0x05 /*PRU0-r30-0	GPIO2_6	  P8-45 R1  SCKL1  PRU1 */
                     0x0a4 0x05 /*PRU1-r30-1	GPIO2_7	  P8-46 R2  SDO1_0 PRU1 */
                     0x0ac 0x05 /*PRU1-r30-3	GPIO2_9	  P8-44 R4  SDO1_1 PRU1 */
                     0x0b0 0x05 /*PRU1-r30-4	GPIO2_10  P8-41 T1  SDO1_2 PRU1 */
                     0x0b4 0x05 /*PRU1-r30-5	GPIO2_11  P8-42 T2  SDO1_3 PRU1 */
                     0x0b8 0x05 /*PRU1-r30-6	GPIO2_12  P8-39 T3  SDO1_4 PRU1 */
                     0x0bc 0x05 /*PRU1-r30-7	GPIO2_13  P8-40 T4  SDO1_5 PRU1 */
                   /* OUTPUT  GPIO(mode7) 0x07 pulldown, 0x17 pullup, 0x?f no pullup/down */
    			   /* INPUT   GPIO(mode7) 0x27 pulldown, 0x37 pullup, 0x?f no pullup/down */
    
    			>;
    		 };
          };
       };
    
       fragment@1 {
    		target = <&ocp>;
    		__overlay__ {
    			test_helper: helper {
    				compatible = "bone-pinmux-helper";
    				pinctrl-names = "default";
    				pinctrl-0 = <&pinctrl_Wol_Cube_IO>;
    				status = "okay";
    			};
    		};
    	};
    
      fragment@2 {
         target = <&pruss>;
            __overlay__ {
              status = "okay";
           };
        };
    	
    };

**Kurze Beschreibung**  
**"exclusive use"** sagt nur aus, dass kein Overlay dieses Überschreiben darf.  
**"fragment@0"** enthält nur die I/O Configuration für den Pin-Mux-Controller.   
**"fragment@1"** aktiviert den Pin-Mux-Controller bzw. den Pin-Mux-Helper  
**"fragment@2"** aktiviert das Pru-System   
In den Kommentaren hinter der jeweiligen I/O-Konfiguration in farment@0, befindet sich auch schon die spätere
Belegung, die für die Softwareentwicklung eine Rolle spielt.
 

Das DTO hat die Dateindung dts. In unserem Fall sieht vollständige Dateiname wie folgt aus:    
   `wol-cube_DTO-00A0.dts`  
 **Wichtig:** Vor der Dateiendung muss -00A0 stehen, weil der Capemanager das Overlay sonst nicht finden kann.  

**Compilieren des DTO**  
Nach dem der Inhalt des DTO jetzt fertig ist müssen wir die *.dts zu einer *.dtbo compilieren:  
 
    dtc -O dtb -o wol-cube_DTO-00A0.dtbo -b 0 -@ wol-cube_DTO-00A0.dts

mit mv/cp die Datei nach /lib/firmware verschieben/kopieren. 

Im Anschluss daran muss die Datei noch dem Capemanger bekannt gemacht werden:  
 
    root@beaglebone:/sys/devices/bone_capemgr.*# echo wol-cube_DTO > slots  
    * = je nach Version z.B. '8'    
Über die Console kann mit `cat /sys/devices/bone_capemgr.*/slots` der Zustand des Capemager abgefragt werden:  
    
    root@beaglebone:~# cat /sys/devices/bone_capemgr.*/slots
     0: 54:PF---
     1: 55:PF---
     2: 56:PF---
     3: 57:PF---
     4: ff:P-O-- Bone-LT-eMMC-2G,00A0,Texas Instrument,BB-BONE-EMMC-2G
     5: ff:P-O-- Bone-Black-HDMI,00A0,Texas Instrument,BB-BONELT-HDMI
     6: ff:P-O-- Bone-Black-HDMIN,00A0,Texas Instrument,BB-BONELT-HDMIN
     7: ff:P-O-L Override Board Name,00A0,Override Manuf,wol-cube_DTO
    root@beaglebone:~#

Das Overlay wurde übernommen.   
Über `cat /sys/kernel/debug/pinctrl/44e10800.pinmux/pins ` kann die Konfiguration der GPIO Pins agbefragt werden:  

    root@beaglebone:~# cat /sys/kernel/debug/pinctrl/44e10800.pinmux/pins
    registered pins: 142
    pin 0 (44e10800) 00000031 pinctrl-single
    pin 1 (44e10804) 00000031 pinctrl-single
    pin 2 (44e10808) 00000031 pinctrl-single
    pin 3 (44e1080c) 00000031 pinctrl-single
    pin 4 (44e10810) 00000027 pinctrl-single
    pin 5 (44e10814) 00000027 pinctrl-single
    pin 6 (44e10818) 00000027 pinctrl-single
    pin 7 (44e1081c) 00000027 pinctrl-single
    pin 8 (44e10820) 00000027 pinctrl-single
    pin 9 (44e10824) 00000027 pinctrl-single
    pin 10 (44e10828) 00000027 pinctrl-single
    pin 11 (44e1082c) 00000027 pinctrl-single
    pin 12 (44e10830) 00000006 pinctrl-single
    pin 13 (44e10834) 00000006 pinctrl-single
    pin 14 (44e10838) 00000027 pinctrl-single
    pin 15 (44e1083c) 00000027 pinctrl-single
    pin 16 (44e10840) 00000027 pinctrl-single
    pin 17 (44e10844) 00000027 pinctrl-single
    pin 18 (44e10848) 00000027 pinctrl-single
    pin 19 (44e1084c) 00000027 pinctrl-single
    pin 20 (44e10850) 00000017 pinctrl-single
    pin 21 (44e10854) 00000007 pinctrl-single
    pin 22 (44e10858) 00000017 pinctrl-single
    pin 23 (44e1085c) 00000007 pinctrl-single
    pin 24 (44e10860) 00000017 pinctrl-single
    pin 25 (44e10864) 00000027 pinctrl-single
    pin 26 (44e10868) 00000027 pinctrl-single
    pin 27 (44e1086c) 00000027 pinctrl-single
    pin 28 (44e10870) 00000037 pinctrl-single
    pin 29 (44e10874) 00000037 pinctrl-single
    pin 30 (44e10878) 00000037 pinctrl-single
    pin 31 (44e1087c) 00000037 pinctrl-single
    pin 32 (44e10880) 00000032 pinctrl-single
    pin 33 (44e10884) 00000032 pinctrl-single
    pin 34 (44e10888) 00000037 pinctrl-single
    pin 35 (44e1088c) 00000027 pinctrl-single
    pin 36 (44e10890) 00000037 pinctrl-single
    pin 37 (44e10894) 00000037 pinctrl-single
    pin 38 (44e10898) 00000037 pinctrl-single
    pin 39 (44e1089c) 00000037 pinctrl-single
    pin 40 (44e108a0) 00000005 pinctrl-single
    pin 41 (44e108a4) 00000005 pinctrl-single
    pin 42 (44e108a8) 00000005 pinctrl-single
    pin 43 (44e108ac) 00000005 pinctrl-single
    pin 44 (44e108b0) 00000005 pinctrl-single
    pin 45 (44e108b4) 00000005 pinctrl-single
    pin 46 (44e108b8) 00000005 pinctrl-single
    pin 47 (44e108bc) 00000005 pinctrl-single
    pin 48 (44e108c0) 0000002f pinctrl-single
    pin 49 (44e108c4) 0000002f pinctrl-single
    pin 50 (44e108c8) 0000002f pinctrl-single
    pin 51 (44e108cc) 0000002f pinctrl-single
    pin 52 (44e108d0) 0000002f pinctrl-single
    pin 53 (44e108d4) 0000002f pinctrl-single
    pin 54 (44e108d8) 0000002f pinctrl-single
    pin 55 (44e108dc) 0000002f pinctrl-single
    pin 56 (44e108e0) 0000000f pinctrl-single
    pin 57 (44e108e4) 0000000f pinctrl-single
    pin 58 (44e108e8) 00000027 pinctrl-single
    pin 59 (44e108ec) 00000027 pinctrl-single
    pin 60 (44e108f0) 00000030 pinctrl-single
    pin 61 (44e108f4) 00000030 pinctrl-single
    pin 62 (44e108f8) 00000030 pinctrl-single
    pin 63 (44e108fc) 00000030 pinctrl-single
    pin 64 (44e10900) 00000030 pinctrl-single
    pin 65 (44e10904) 00000030 pinctrl-single
    pin 66 (44e10908) 00000027 pinctrl-single
    pin 67 (44e1090c) 00000027 pinctrl-single
    pin 68 (44e10910) 00000020 pinctrl-single
    pin 69 (44e10914) 00000000 pinctrl-single
    pin 70 (44e10918) 00000020 pinctrl-single
    pin 71 (44e1091c) 00000000 pinctrl-single
    pin 72 (44e10920) 00000000 pinctrl-single
    pin 73 (44e10924) 00000000 pinctrl-single
    pin 74 (44e10928) 00000000 pinctrl-single
    pin 75 (44e1092c) 00000020 pinctrl-single
    pin 76 (44e10930) 00000020 pinctrl-single
    pin 77 (44e10934) 00000020 pinctrl-single
    pin 78 (44e10938) 00000020 pinctrl-single
    pin 79 (44e1093c) 00000020 pinctrl-single
    pin 80 (44e10940) 00000020 pinctrl-single
    pin 81 (44e10944) 00000027 pinctrl-single
    pin 82 (44e10948) 00000030 pinctrl-single
    pin 83 (44e1094c) 00000010 pinctrl-single
    pin 84 (44e10950) 00000037 pinctrl-single
    pin 85 (44e10954) 00000037 pinctrl-single
    pin 86 (44e10958) 00000062 pinctrl-single
    pin 87 (44e1095c) 00000062 pinctrl-single
    pin 88 (44e10960) 0000002f pinctrl-single
    pin 89 (44e10964) 00000027 pinctrl-single
    pin 90 (44e10968) 00000037 pinctrl-single
    pin 91 (44e1096c) 00000037 pinctrl-single
    pin 92 (44e10970) 00000030 pinctrl-single
    pin 93 (44e10974) 00000000 pinctrl-single
    pin 94 (44e10978) 00000073 pinctrl-single
    pin 95 (44e1097c) 00000073 pinctrl-single
    pin 96 (44e10980) 00000037 pinctrl-single
    pin 97 (44e10984) 00000037 pinctrl-single
    pin 98 (44e10988) 00000070 pinctrl-single
    pin 99 (44e1098c) 00000070 pinctrl-single
    pin 100 (44e10990) 00000005 pinctrl-single
    pin 101 (44e10994) 00000005 pinctrl-single
    pin 102 (44e10998) 00000005 pinctrl-single
    pin 103 (44e1099c) 00000005 pinctrl-single
    pin 104 (44e109a0) 00000024 pinctrl-single
    pin 105 (44e109a4) 00000005 pinctrl-single
    pin 106 (44e109a8) 00000027 pinctrl-single
    pin 107 (44e109ac) 00000005 pinctrl-single
    pin 108 (44e109b0) 00000023 pinctrl-single
    pin 109 (44e109b4) 00000027 pinctrl-single
    pin 110 (44e109b8) 00000030 pinctrl-single
    pin 111 (44e109bc) 00000028 pinctrl-single
    pin 112 (44e109c0) 00000030 pinctrl-single
    pin 113 (44e109c4) 00000028 pinctrl-single
    pin 114 (44e109c8) 00000028 pinctrl-single
    pin 115 (44e109cc) 00000028 pinctrl-single
    pin 116 (44e109d0) 00000030 pinctrl-single
    pin 117 (44e109d4) 00000030 pinctrl-single
    pin 118 (44e109d8) 00000030 pinctrl-single
    pin 119 (44e109dc) 00000030 pinctrl-single
    pin 120 (44e109e0) 00000020 pinctrl-single
    pin 121 (44e109e4) 00000030 pinctrl-single
    pin 122 (44e109e8) 00000030 pinctrl-single
    pin 123 (44e109ec) 00000028 pinctrl-single
    pin 124 (44e109f0) 00000028 pinctrl-single
    pin 125 (44e109f4) 00000028 pinctrl-single
    pin 126 (44e109f8) 00000030 pinctrl-single
    pin 127 (44e109fc) 00000028 pinctrl-single
    pin 128 (44e10a00) 00000028 pinctrl-single
    pin 129 (44e10a04) 00000020 pinctrl-single
    pin 130 (44e10a08) 00000028 pinctrl-single
    pin 131 (44e10a0c) 00000028 pinctrl-single
    pin 132 (44e10a10) 00000028 pinctrl-single
    pin 133 (44e10a14) 00000028 pinctrl-single
    pin 134 (44e10a18) 00000028 pinctrl-single
    pin 135 (44e10a1c) 00000020 pinctrl-single
    pin 136 (44e10a20) 00000028 pinctrl-single
    pin 137 (44e10a24) 00000028 pinctrl-single
    pin 138 (44e10a28) 00000028 pinctrl-single
    pin 139 (44e10a2c) 00000028 pinctrl-single
    pin 140 (44e10a30) 00000028 pinctrl-single
    pin 141 (44e10a34) 00000020 pinctrl-single

**DTO beim Systemstart**  
Damit das DTO beim Systemstart direkt geladen wird, muss es ebefalls in die **uEnv.txt** eingetragen werden,  
in der schon das HDMI disabled wurde:

    optargs=quiet capemgr.disable_partno=BB-BONELT-HDMI,BB-BONELT-HDMIN,BB-BONE-EMMC-2G capemgr.enable_partno=wol-cube_DTO   

Jetzt ist das BBB soweit vorbereitet, dass mit der Softwareentwicklung begonnen werden kann.  


    



  
 

 


 

        
  
 
 

      
  
 



       


    