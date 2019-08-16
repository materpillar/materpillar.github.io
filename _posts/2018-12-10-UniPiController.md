---
layout: post
title:  "Making a UniFi Conroller from a Raspberry Pi 2B+"
date:   2018-12-10 22:00:00 +0200
categories: project
---


# Vorgeschichte

Seit irgendwann Mitte 2015 bin ich Besitzer eines Raspberry Pi 2 Model B. Diesen habe ich über die Jahre insbesondere als Mediencenter mit Kodi benutzt.
Zwischenzeitlich hatte ich daran auch einen Infrarotempfänger um ihn mit einer alten Fernsehfernbedienung benutzen zu können. Irgendwann ist das Projekt dann eingeschlafen.
Das lag an verschiedenen Problemchen
* Das erste Netzteil, das ich verwendet habe, hat ohne Last unglaublich gefiept (3-Ampere-Billig-Ladegerät importiert...)
* Das Netzteil, das ich als Ersatz von Conrad bestellt hatte (MeanWell) lieferte anstatt der versprochenen 3 Ampere so wenig, dass selbst ohne angeschlossene USB-Festplatte der Pi bereits Strommangel meldete.
* Die Integration eines ordentlichen Schalters und LEDs in ein umgebautes Linkstation Pro Case wurde sehr aufwendig und ab einem bestimmten Punkt nicht mehr fortgesetzt.
* Ein Case das ich selber bauen wollte, wurde leider nie realisiert.

Irgendwann kam das neue Raspberry Model mit WLAN und dann sogar irgendwann mit Gigabit-Ethernet und PoE, sodass ich ein zukünftiges Mediencenter nur damit umsetzen wollte.
Doch was tun mit dem vorhandenen 2er Model?

Durch mein Praktikum hatte ich einige Skills in Sachen Löten und generelle Elektronik-Eigenbau erworben. Das Zuhause meiner Eltern hatten wir schon vor einiger Zeit zu einem Power-over-Ethernet-Netzwerk aufgerüstet, um Switche und Access-Points auf allen Stockwerken ohne lästige Netzkabel einbauen zu können.
Das Netzwerk basiert dabei auf Komponenten von Unifi. Diese benötigen (zumindest zur Einrichtung) einen PC mit installiertem Controller. Übergangsweise wurde der auf dem PC meines Vaters installiert. Das hatte allerdings den Nachteil, dass ich immer an diesen musste, wenn ich etwas umstellen wollte.
Auch ein Ausfall des PCs könnte eventuell Probleme bereiten.

So entstand die Idee einen eigenen UnifiCloud Controller aus dem alten Raspberry zu basteln. Natürlich mit PoE-Unterstützung.
Ich möchte hier darauf hinweisen, dass das umgesetzte PoE keinem Standard entspricht! Es funktioniert mit passiven 24V PoE, welcher sich in unserem Switch so konfiguieren lies.

Die urspünglichste Idee zur Schaltung und zum Aufbau habe ich von hier:
https://www.instructables.com/id/PiPoE-powering-a-Raspberry-Pi-over-Ethernet/


# Den Raspberry Pi 2B+ PoE-fähig machen



24 Volt passive PoE-fähig


# Software

## Rasbpian Lite

### username pi change

### unattended-upgrade
https://wiki.debian.org/UnattendedUpgrades


## unifi installation

Install java-8 first!
sudo apt-get install openjdk-8-headless

https://help.ubnt.com/hc/en-us/articles/220066768-UniFi-How-to-Install-and-Update-via-APT-on-Debian-or-Ubuntu

### Adding the repository

### Configuring