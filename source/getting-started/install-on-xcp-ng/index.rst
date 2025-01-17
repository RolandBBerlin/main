.. _install-on-xen-label:

============================
 Virtualisierung mit XCP-ng
============================

.. sectionauthor:: `@cweikl <https://ask.linuxmuster.net/u/cweikl>`_

XCP-ng ist eine reine OpenSource-Virtualisierungslösung, die auf Basis 
von XEN arbeitet. XCP-ng bietet sog. Enterprise-Features wie Replikation, 
automatisierte Backups, Verschieben von VMs im laufenden Betrieb und 
weitere Funktionen. Daher eignet sie sich besonders für den virtuellen 
Betrieb von linuxmuster.net, da diese recht einfach skalierbar ist,
mehrere Virtualisierungs-Hosts in einem sog. Ressource-Pool zusammengeführt
und verwaltet werden können.

Der Betrieb wird auf jeglicher Markenhardware und auf einer Vielzahl an 
NoName-Hardware unterstützt.

In diesem Dokument findest Du "Schritt für Schritt" Anleitungen zum
Installieren der linuxmuster.net-Musterlösung in der Version 6.2 auf
Basis von XCP-ng.

Nach der Installation gemäß dieser Anleitung erhältst Du eine
einsatzbereite Umgebung bestehend aus

* einem Host (XCP-ng) für alle virtuellen Maschinen, 
* einer Firewall (IPFire)  
* einem Server (linuxmuster.net v6.2)
* einer VM (XOA) zur web-basierten Verwaltung des Virtualisierungs-Hosts
* Optional: einer VM zur Softwareverteilung für Windows-Clients OPSI

Voraussetzungen
===============

* Es wird vorausgesetzt, dass Du einen Administrationsrechner
  (Admin-PC genannt) besitzt, den Du je nach Bedarf in die
  entsprechenden Netzwerke einstecken kannst und dessen
  Netzwerkkonfiguration entsprechend vornehmen kannst.

* Der Internetzugang des Admin-PCs und auch des XCP-ng-Hosts sollte
  zunächst gewährleistet sein, d.h. dass beide zunächst z.B. an einem
  Router angeschlossen werden, über den die beiden per DHCP oder fester IP 
  ins Internet können. Sobald später die Firewall korrekt eingerichtet
  ist, bekommt der Admin-PC und bei Bedarf auch der XCP-ng-Host eine
  IP-Adresse im Schulnetz.

.. hint:: 

   Virtualisierungs-Hosts sollten grundsätzlich niemals im gleichen Netz wie 
   andere Geräte sein, damit dieser nicht von diesen angegriffen werden kann.
   In dieser Dokumentation wird zur Vereinfachung der Fall dokumentiert, dass
   der XCP-ng-Host sich im internen Schulnetz befindet.

Bereitstellen des XCP-ng-Hosts
==============================

.. hint:: 

   Der XCP-ng-Host bildet das Grundgerüst für die Firewall *IPFire* und
   den Schulserver *server*. Die Virtualisierungsfunktionen der CPU sollten 
   zuvor im BIOS aktiviert worden sein.

Die folgende Anleitung beschreibt die *einfachste* Implementierung
ohne Dinge wie VLANs, Teaming oder RAIDs. Diese Themen werden in
zusätzlichen Anleitungen betrachtet.


Download-Quellen VMs
--------------------

Die Quellen finden Sie im Abschnitt 

:ref: `Voraussetzungen - Downloads <getting-started-downloads-label>`_ 

detailliert beschrieben.

Alternativ finden sich alle VMs zum Download auf der Download-Übersicht 
XCP-Übersicht-v6.2_:

.. _XCP-Übersicht-v6.2: https://download.linuxmuster.net/xcp-ng/v6.2/


Erstellen eines USB-Sticks zur Installation des XCP-ng-Host
-----------------------------------------------------------

Für die Installation wird benötigt:

* ein Installationsdatenträger mit XCP-ng


Installation XCP-ng
===================

Herunterladen von XCP-ng
------------------------
Diese Anleitung bezieht sich auf die Version 7.6. Für nachfolgende Versionen ist 
dieses Vorgehen entsprechend anzuwenden.

Die ISO-Datei muss heruntergeladen und ein bootfähiger USB-Stick erstellt werden.

1. Herunterladen: XCP-Webseite_

.. _XCP-Webseite: https://xcp-ng.org/#easy-to-install

2. USB-Stick erstellen: In das Download-Verzeichnis wechseln, Buchstaben für 
USB-Stick unter Linux ermitteln, X durch den korrekten Buchstaben ersetzen und 
dann nachstehenden Befehl eingeben:

.. code-block:: console
 
   dd if=XCP-ng_7.6.0.iso of=/dev/sdX bs=8M status=progress oflag=direct


Installieren von XCP-ng
-----------------------

Vom USB-Stick booten, danach erscheint folgender Bildschirm:

.. figure:: media/01_xcp-ng-install.png
   :align: center
   :alt: Splash Screen

Starten der Installtion mit ``ENTER``.

Wähle das Tastaturlayout.

.. figure:: media/02_xcp-ng-install_select-keymap.png
   :align: center
   :alt: Auswahl des Tastatur-Layout

Wir verwenden ``[querz] de``.

Sollten zusätzliche Treiber benötigen werden können diese nun mit ``F9`` geladen werden.
Starte das XCP-ng Setup mit ``Ok``.

.. figure:: media/03_xcp-ng-install_welcome.png
   :align: center
   :alt: Setup Begrüßunsbildschirm

Akzeptiere die Lizenzbedingungen mit ``Accept EULA``.

.. figure:: media/04_xcp-ng-install_eula.png
   :align: center
   :alt: End User License Agreement

XCP-ng prüft die verwendete System Hardware. Eventuell wird die Nichtaktivierung der Hardware Virtualisierung beanstandet. Bestätige die Meldung und schalten die Hardware Virtualisierung später im BIOS ein.

.. figure:: media/05_xcp-ng-install_system-hardware.png
   :align: center
   :alt: Probleme mit der Hardware

Bei einer Neuinstallation werden für das gewählte Medium dann die Partitionen erstellt und das Dateisystem erzeugt. 

.. figure:: media/06_xcp-ng-install_virtual-machine-storage.png
   :align: center
   :alt: Speicherplatz-Einrichtung des Virtual-Hosts

Danach wird nach der Installationsquelle gefragt.

.. figure:: media/07_xcp-ng-install_select-installation-source.png
   :align: center
   :alt: Installationsmedium

Gebe hier ``Local Media`` an.

Danach wirst Du gefragt, ob das Installationsmedium überprüft werden soll.

.. figure:: media/08_xcp-ng-install_verify-installation-source.png
   :align: center
   :alt: Überprüfung des Installationsmediums

Bestätige dies mit ``Verfy installation source``.

Nach Abschluss der erfolgreichen Überprüfung des Installationsmediums wird dies bestätigt.

.. figure:: media/09_xcp-ng-install_verification-sucessful.png
   :align: center
   :alt: Installationsmedium in Ordnung

Lege danach das Kennwort für den Administrator (user: root) fest und bestätigen dieses.

.. figure:: media/10_xcp-ng-install_set-password.png
   :align: center
   :alt: Passwort des root-Accounts

Nun werden die Netzwerkeinstellungen abgefragt.
Festlegen der IP-Adresse über die der Virtualisierer im Netz erreichbar ist.

.. figure:: media/11_xcp-ng-install_networking.png
   :align: center
   :alt: Netzwerk des Virtual-Hosts

Hostnamen festlegen und die DNS-Server eintragen.

.. figure:: media/12_xcp-ng-install_hostname-and-dns-configuration.png
   :align: center
   :alt: Hostname des Virtual-Hosts und die IPs der lokalen DNS

Erst die Region wählen.

.. figure:: media/13_xcp-ng-install_select-time-zone.png
   :align: center
   :alt: Auswahl des Standorts der Installation - Region

Danach die Stadt auswählen.

.. figure:: media/14_xcp-ng-install15.png
   :align: center
   :alt: Zeitzone des Standorts

Zeit manuel eintragen auswählen

.. figure:: media/15_xcp-ng-install_system-time.png
   :align: center
   :alt: Manuelle Eingabe der Zeit


Bestätige danach die Frage nach der Installation von XCP-ng.

.. figure:: media/16_xcp-ng-install_confirm-installation.png
   :align: center
   :alt: Starten des Installation-Vorganges

Danach startet die Installation

.. figure:: media/17_xcp-ng-install_installing-scp-ng.png
   :align: center
   :alt: Vorbereiten der Installation

Die Frage nach Installation eines ``Supplemental Pack`` ist mit ``No`` zu beantworten.

.. figure:: media/18_xcp-ng-install_supplemental_packs.png
   :align: center
   :alt: Keine Installation von Supplemental Packs

Installation wird fortgesetzt.

.. figure:: media/19_xcp-ng-install_installing.png
   :align: center
   :alt: Fertigstellung der Installation

Uhrzeit und anders einstellen und mit Enter bestätigen.

.. figure:: media/20_xcp_ng-install_set-local-time.png
   :align: center
   :alt: Einstellen von Datum und Uhrzeit

Nach erfolgreicher Installation kannSt Du mit ``Ok`` den Server neu starten.
Achte darauf, dass der USB-Stick nicht mehr für den Bootvorgang aktiv ist.

.. figure:: media/21_xcp-ng-install_installation-complete.png
   :align: center
   :alt: Installation abgeschlossen

Beim Startvorgang erscheint folgende Auswahl:

.. figure:: media/22_xcp-ng-install_grub-menu.png
   :align: center
   :alt: GRUB Start-Menu

XCP-ng wird nach einigen Sekunden automatisch gestartet.

.. figure:: media/23_xcp-ng-install_start-screen.png
   :align: center
   :alt: Splash-Screen des Virtual Hosts

Nach erfolgreichem Start bootet XCP-ng in folgende Konsole des Hypervisors:

.. figure:: media/24_xcp-ng-install_dash.png
   :align: center
   :alt: Konsole des Virtual Hosts


Aktualisierung des XCP-ng-Hosts
-------------------------------

Wähle in dem Startbildschirm des XCP-ng Hosts den Menüpunt ``Local Command Shell``
und drücke ``Enter``. Gebe als Benutzer ``root`` an und das Passwort das Du 
während der Installation vergeben hast.

.. figure:: media/25_xcp-ng-install_update.png
   :align: center
   :alt: Update des Virtual Hosts

Gebe auf der Konsole den Befehl 

.. code-block:: console
 
   yum update

ein. XCP-ng fragt nun via Internetverbindung die Repositories ab und prüft, ob
Aktualisierungen vorhanden sind. Falls ja, werden die zu aktualisierenden Pakete 
angezeigt. Die Aktualisierung ist mit ``y`` zu starten.

Danach ist Dein XCP-ng Host auf dem aktuellen Stand.

XCP-ng: Administration
=======================

Für die Administration deines XCP-ng-Hosts stehen Dir zwei Möglichkeiten zur Verfügung.

Der XCP-ng-Host kann web-basiert administriert werden.
Dies erfolgt mithilfe der Anwendung XenOrchestra (XOA - Xen Orchestra Application).
linuxmuster.net stellt hierfür ebenfalls eine vorkonfigurierte VM mit einer installierten XOA App zur Verfügung. XOA wurde hier "from stratch" installiert und an die lmnv6.2 angepasst.
Eine Beschreibung befindet sich weiter unten in dieser Anleiltung.

Die andere Möglichkeit nutzt einen Adminstrations-Rechner.
Auf diesem installierst Du Dir auf einem Rechner im Netzwerk das Windows-Programm ``XCP-ng Center``.
Hiermit kannst Du die gesamte Umgebund administrieren und insbesondere die vorkonfigurierten VMs einfach importieren.
Wir beschreiben hier die Installation und Benutzung unter Windows.

Eine Anleitung für die Installation des Programms unter Linux mithilfe von Wine und PlayOnLinux wird unter :ref:`XCP-ng Center unter Linux installieren <XCP-ng-Center-Linux-label>` beschrieben.  


XCP-ng Center unter Windows installieren
----------------------------------------

Lade Dir das Windows-Programm zur Verwaltung von der Seite des XCP-ng Projekts herunter:

XCP-ng Center AktuelleVersion_

.. _AktuelleVersion: https://github.com/xcp-ng/xenadmin/releases

Installiere das Programm durch einen Rechtsklick auf die MSI-Datei auf dem Windows-Rechner und wähle dann ``Als Administrator ausführen`` aus.

.. figure:: media/26_xcp-ng-admin_start.png
   :align: center
   :alt: XCP-ng Center Installation

Bestätige die Rückfrage mit ``Ja``

.. figure:: media/27_xcp-ng-admin_authorisation.png
   :align: center
   :alt: Windows Benutzerkonensteuerung

Rufe nach erfolgreicher Installation das Programm ``XCP-ng Center`` auf.

Wähle hier den Menüpunkt ``Add New Server`` 

.. figure:: media/28_xcp-ng_open-add-server.png
   :align: center
   :alt: Hinzufügen des Virtual Hosts

Gebe die bei der Installation vergebene IP-Adresse des XCP-Hosts sowie die Benutzerdaten an.

.. figure:: media/29_xcp-ng-admin_ip_and_username.png
   :align: center
   :alt: IP oder Name des Virtual Hosts und Benutzerdaten

Das folgende Fenster kann mit Klick auf „Close“ geschlossen werden.

.. figure:: media/30_xcp-ng-admin_health-check.png
   :align: center
   :alt: Überspringen des Checks


Netzwerk einrichten
~~~~~~~~~~~~~~~~~~~

Jetzt muss das Netzwerk eingerichtet werden. Notiere Dir hierzu die Bezeichnungen
und MAC-Adressen der eingebauten Netzwerkkarten. Diese findest Du unter der Reiterkarte ``NICs``.
Die Netzwerkkarte, die die Verbindung zum Internet übernehmen soll wird später dem Netzwerk ``Red``, 
diejenige für das interne Schulungsnetz dem Netzwerk ``Green`` und die dritte Netzwerkkarte 
für die Steuerung des WLAN dem Netzwerk ``Blue`` zugeordnet.

Damit dies korrekt erfolgt, ist es wichtig zu wissen, wie NIC 0,1,2 physikalisch angeschlossen sind
und welche MAC-Adressen diese aufweisen. Anhand der Informationen erfolgt dann im folgenden Schritt
die Zuordnung der Netze (vSwitche).

Wähle nun Für den XCP-ng-Host die Reiterkarte ``Networking`` aus.

.. figure:: media/31_xcp-ng-admin_networking.png
   :align: center
   :alt: Reiter Networking

Wähle das erste Netwerk ``Network 0`` aus, prüfe die Zurdonung der Netzwerkkarte. 

.. figure:: media/32_xcp-ng-admin_network-0.png
   :align: center
   :alt: Auswahl der Netzwerke

Es muss diejenige zugewiesen sein, die die Internet-Verbindung steuert. Klicke dann auf ``Properties`` 
und ändere den Namen für das Netzwerk in ``RED``.

.. figure:: media/33_xcp-ng-admin_network-rename.png
   :align: center
   :alt: Umbenennung der Netzwerks

Führe diese Schritte ebenfalls für die weitere Netze aus und ändere die Namen auf ``BLUE`` und ``GREEN``.

.. figure:: media/34_xcp-ng-admin_networks-renamed.png
   :align: center
   :alt: Fertig eingerichtete Netzwerke

VMs importieren
~~~~~~~~~~~~~~~

Nachdem das Netzwerk korrekt eingerichtet wurde, können nun die VMs der linuxmuster.net 
importiert werden.

Lade Dir vorher zunächst alle VMs, die Du importieren möchtest unter linuxmuster.net herunter.

Danach rufe im XCP-ng Center den Menüpunkt ``File -> Import`` auf.

.. figure:: media/35_xcp-ng-import_menue-import.png
   :align: center
   :alt: Menu um VMs zu importieren

Es erscheint ein neues Fenster.

.. figure:: media/36_xcp-ng-import_import-window.png
   :align: center
   :alt: Import Source

Mittels ``Browse ...`` wählst Du erst den Speicherort aus.

.. figure:: media/37_xcp-ng-import_import-browse2file.png
   :align: center
   :alt: Auswahl des Speicherorts

Dann die Datei mit der zu importierenden VM.

.. figure:: media/38_xcp-ng-import_select-importfile.png
   :align: center
   :alt: Auswahl der zu importierenden VM

Die VMs weisen die Dateiendung ``.xva`` auf.

Nach Bestätigung mit ``Öffnen`` erscheint nun das erste Fenster, um den Import zu steuern.

.. figure:: media/39_xcp-ng-import_file2import.png
   :align: center
   :alt: Ausgewählte VM 

Zunächst must Du den XCP-ng-Host festlegen, für den der Import der VM erfolgen soll.

.. figure:: media/40_xcp-ng-import_home-server.png
   :align: center
   :alt: Virtual Host für die zu importierende VM

Wähle danach Deinen gewünschten Speicher aus. Bestätige mit ``Next``.

.. figure:: media/41_xcp-ng-import_storage.png
   :align: center
   :alt: Speicher für die zu importierende VM

Prüfe die Netzwerkeinstellungen, die von der zu importierenden VM stammen.

.. figure:: media/42_xcp-ng-import_networking.png
   :align: center
   :alt: Die NEtzwerke für die zu importierende VM

Bestätige diese mit ``Next``.

Prüfe nun nochmals alle Einstellungen für den Import der VM.
Falls Änderungen erforderlich sind, gehe mit ``Previous`` zurück zur gewünschten Einstellung.

.. figure:: media/43_xcp-ng-import_finish.png
   :align: center
   :alt: Zusammenfassung aller Import-Daten

Bestätige nun den Import mit ``Finish``.

Der Import kann einige Zeit dauern. Danach solltest Du die importierte 
VM im XCP-ng Center sehen können.

.. figure:: media/44_xcp-ng-import_imported-vms.png
   :align: center
   :alt: Übersicht über die importierten VMs


VMs starten und aktualisieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Wähle im XCP-ng Center links die VM aus, die Du starten möchtest.
Klicke danach oben in der Menüleiste das Icon ``Start`` aus.

Beginne mit der Firewall IPFire. Starte diese.

.. figure:: media/45_xcp-ng-import_ipfire-started.png
   :align: center
   :alt: IPFire - Konsole gestartet 

Starte die VM mit dem linuxmuster.net Server.

.. figure:: media/46_xcp-ng-import_lmn-server-started.png
   :align: center
   :alt: lmn-Server - Konsole gestartet

Sofern Du weitere VMs importiert hast, starte diese.

.. _XCP-ng-Center-Linux-label:

XCP-ng Center unter Linux installieren
--------------------------------------

XCP-ng Center ist eine Anwendung zur Administration des XCP-ng Virtualisierers, 
die für den Betrieb unter Windows programmiert wurde. Um diese Verwaltungssoftware 
betriebssystemunabhängig einzusetzen, nutzt Du die bereits vorkonfigurierte 
virtuelle Maschine (VM) Xen Orchestra (XOA) und importierst diese in XCP-ng. 

Weitere Hinweise findest Du unter 'Xen Orchestra (XOA)`_

Für die Installtion unter Linux sind folgende Schritte notwendig:

1. Installation einer aktuellen Wine Version unter Linux
2. Installation von PlayOnLinux
3. INstalation der aktuellen XCP-ng Center App via PlayOnLinux Plugin
4. Verbindung zum XCP-ng Server via Port 80


Installation von Wine
~~~~~~~~~~~~~~~~~~~~~

Zunächst muss Wine für das jeweils genutzte Linux-Derivat installiert werden. 
Das Projekt ``Wine`` bietet hierzu eine Reihe an Hinweisen an. 
Diese stehen ebenfalls für die jeweiligen Linux-Derivate zur Verfügung:

- https://wiki.winehq.org/Wine_Installation_and_Configuration
- https://wiki.winehq.org/Debian
- https://wiki.debian.org/Wine
- https://wiki.winehq.org/Ubuntu

Hast Du für Dein Linux Wine installiert, ist nun PlayOnLinux zu installieren.

Installation PlayOnLinux
~~~~~~~~~~~~~~~~~~~~~~~~

Für die jeweiligen Linux-Derivate stehen fertige Pakete für die Installation zur 
Verfügung. Diese finden sich inkl. den Installationshinweisen unter InstPlayOnLinux_:

.. _InstPlayOnLinux: https://www.playonlinux.com/en/download.html

In der Regel verfügen die Linux-Derivate bereits über eingetragene Paketquellen 
für PlayOnLinux. Über den Download-Bereich des Projekts sind die aktuellsten Pakete 
zu erhalten.

.. hint::

   Es sollte wine 4.0 (i386) mit 32-Bit Unterstützung und PlayOnLinux 4.3.4 installiert 
   sein. PlayOnLinux soll Windows 7 simulieren.


Installation von XCP-ng Center
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Für die Installation von XCP-ng Center must Du vorab eine XCP-ng Center Version
herunterladen, die für die Installation mit PlayOnLinux vorbereitet wurde. Es handelt
sich hierbei um einen PlayOnLinux Container, der XCP-ng Center mit allen Abhängigkeiten 
(IE8, .NET Framework 2.0 SP2 und .NET Framework 4.7.2) enthält.

Die aktuellste Version_ lädst Du vorab herunter:

.. _Version: https://github.com/aldebaranbm/xencenter-playonlinux/releases/tag/2019-02-05

Danach rufst Du PlayOnLinux auf. Dort gehst Du im Menü auf den 
``Menüpunkt -> Erweiterungen (Plugins) -> Untermenü PlayOnLinux Vault``.

Es erscheint dann ein neues Fenster für die weitere Installation der Anwendung.

.. figure:: media/47_center-on-linux_playonlinux-vault.png
   :align: center
   :alt: Start der Installation mittels PlayOnLinux

Klicke hier auf ``Weiter``.

Du gelangst zum nächsten Fenster, in dem Du angegeben kannst, ob Du eine Anwendung installieren
oder deinstallieren möchtest.

.. figure:: media/48_center-on-linux_restore-an-application.png
   :align: center
   :alt: Installieren einer Anwendung

Wähle hier die Option ``Restore an applications...`` 
und gehe auf ``Weiter``.

Im nächsten Schritt must Du die Anwendung angeben, die zu installieren ist. 

.. figure:: media/49_center-on-linux_choose-the-application.png
   :align: center
   :alt: Suchen der Anwendung

Hier must Du auf ``Durchsuchen`` klicken und dann im Dateisystem den bereits
heruntergeladenen PlayOnLinux-Container mit XCP-ng Center angeben. Die Datei 
weist die Dateierweiterung ``.polApp`` auf.

.. figure:: media/50_center-on-linux_choose-xencenter-dot-polapp.png
   :align: center
   :alt: Datei xencenter.polApp auswählen

Danach klickst Du auf ``Weiter``.

.. figure:: media/51_center-on-linux_data-of-the-app.png
   :align: center
   :alt: Zusammenfassung der Installation 

Es wird nochmals eine Übersicht angezeigt, mit der zu installierenden Anwendung
und dem erforderlichen Speicherplatz.

.. figure:: media/51_center-on-linux_data-of-the-app.png
   :align: center
   :alt: Überprüfung der Installations-Daten

Klicke für die Installation auf ``Weiter``.

Der Installationfortschritt wird Dir angezeigt.

.. figure:: media/52_center-on-linux_installation-progress.png
   :align: center
   :alt: Fortschrittsanzeige

Nach erfolgreicher Installtion siehst Du folgendes Fenster:

.. figure:: media/53_center-on-linux_installation-successful.png
   :align: center
   :alt: Installation erfolgreich

Gehe auf ``Weiter``. Das Fenster wird dadurch geschlossen.


Aufruf XCP-ng Center unter PlayOnLinux
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Die zuvor installierte XCP-ng Anwendung findest Du nun unter PlayOnLinux.

.. figure:: media/54_center-on-linux_is-located.png
   :align: center
   :alt: Starten von XCP-ng Center 

Markiere die Anwendung und gehe links im Kontextmenü auf ``Ausführen``.

Das Programm startet dann.

Greife nun auf XCP-ng zu, indem zu als Server die IP + Portnummer angibst.
Es funktioniert derzeit nur der Port 80. Ein Zugriff auf Port 443 ist derzeit 
noch nicht möglich.

.. figure:: media/55_center-on-linux_add-new-server.png
   :align: center
   :alt: Hinzufügen des XCP-ng Hosts

Gebe hier die lokale IP des XCP-Hosts dann einen Doppelpunkt und die Portnummer an. 
Z.B. ``192.168.199.59:80``

.. note::
   Es erfolgt somit kein verschlüsselter Zugriff auf den XCP-Host. Bitte unbedingt beachten !

.. figure:: media/56_center-on-linux_started_xcp-ng-center.png
   :align: center
   :alt: Zugriff auf den Host 

Um später XCP-ng unter Linux direkt vom Desktop aus aufrufen zu können, kannst Du in PlayOnLinux
XCP-ng als Anwendung in der rechten Hälfte des Fenster markieren und links dann im 
Kontextmenü den Eintrag ``Eintrag erstellen`` auswählen.

Danach findet sich auf dem Desktop der gewünschte Starter-Eintrag.


Mögliche Fehler mit PlayOnLinux
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sollte nach Aufruf des Programm mit PlayOnLinux ein Fehlerfenster erscheinen,
so gibt es verschiedene Fehlerquellen.

.. figure:: media/57_center-on-linux_error-message.png
   :align: center
   :alt: Fehler bei Starten von XCP-ng Center unter PlayOnLinux

Es ist häufiger der Fall, dass Wine in einer 64-Bit Umgebung installiert wurde und 
nur 64-Bit Programme lauffähig sind. XCP-ng Center benötigt allerdings 32-Bit 
Laufzeitumgebungen für Wine.

.. figure:: media/58_center-on-linux_needs-32-bit.png
   :align: center
   :alt: Mögliche Fehlerursache: Fehlende 32-Bit Unterstürtzung

In diesem Fall kannst Du einfach wine32 nachinstallieren, indem Du root 
auf der Eingabekonsole für Debian - Derivate angibst:

  sudo apt-get install wine32

Sollten danach immer noch Fehler auftreten, so solltest Du
die Wine-Istallation und die PlayOnLinux - Installation aktualisieren_.

.. _aktualisieren: http://tipsonubuntu.com/2019/02/01/install-wine-4-0-ubuntu-18-10-16-04-14-04/

Sollte es weiterhin Probleme geben, so must Du ggf. einen Rebuild erstellen. 
Hinweise hierzu erhälst Du auf GitHub unter_

.. _unter: https://github.com/aldebaranbm/xencenter-playonlinux


Xen Orchestra Appliance(XOA)
----------------------------

Xen Orchestra Appliance (XOA_) bietet die Möglichkeit, die Virtualisierungsumgebung XCP-ng webbasiert und plattformunabhängig zu administrieren. Die bereitgestellten
Funktionen entsprechen denen des Programms XCP-ng Center für Windows und gehen hinsichtlich der Backups darüber hinaus. Es können via Borwserzugriff VMs importiert, 
exportiert, neue VMs erstellt und verschoben werden. Zudem lassen sich so plattformunabhängig verschiedene Arten von Backups auf unterschiedlichen Datenträgern erstellen
und Zeitpläne zur automatisierten Erstellung der Backups definieren und aktivieren. 

.. _XOA: https://xen-orchestra.com

Xen Orchestra wird von der französischen Firma vates_ entwickelt und supportet. Diese stellt XOA als Open Source zur Verfügung. Der Quellcode findet sich auf github_.

.. _vates: https://vates.fr/

.. _github: https://github.com/vatesfr/xen-orchestra

linuxmuster.net hat gemäß dieser Anleitung_ eine XOA-VM zum Einsatz auf der Virtualisierungsumgebung XCP-ng auf Basis von Ubuntu 18.04 LTS mit Anpassungen für 
linuxmuster v7 erstellt. Die VM wurde ``from the sources`` erstellt, und für den Betrieb mit linuxmuster.net auf XCP-ng angepasst.

.. _Anleitung: https://xen-orchestra.com/docs/from_the_sources.html

.. note::
 Um XOA VM nutzen zu können, muss diese zuerst unter XCP-ng importiert worden sein!


Import der VM
~~~~~~~~~~~~~

Lade zuerst die vorbereitete XOA-VM für linuxmuster.net als ZIP-Archiv_ herunter. Entpacke dieses Archiv lokal (ca. 6 GiB) und importiere dann die VM wie bereits zuvor 
im Unterkapitel_ ``VMs importieren`` beschrieben.  

.. _ZIP-Archiv: http://fleischsalat.linuxmuster.org/xva/lmn7-xoa-2019-03-08.zip

.. _Unterkapitel: http://docs.linuxmuster.net/de/v7/appendix/install-on-xcp-ng/index.html#vms-importieren

Anpassung der VM
~~~~~~~~~~~~~~~~

Einige Einstellungen der vorkonfigurierten VM sind nach dem Import auf die eigene Virtualisierungsumgebung anzupassen. Öffne hierzu einen Webbrowser und öffne die Seite 
http://10.16.1.4 oder https://10.16.1.4. Der PC, auf dem der Browser geöffnet wird, muss sich im Netz 10.16.0.0/12 (grünes Netz - internes LAN der linuxmuster.net) befinden,
damit eine Verbindung möglich ist. Wählst Du den verschlüsselten Zugriff, so bestätige die Zertifikatswarnung, da ein selbst erstelltes Zertifikat für XOA ertsellt und 
konfiguriert wurde.

Es erscheint folgende Anmeldemaske:
 
.. figure:: media/59_xoa_vm-https-login.png
   :align: center
   :alt: Anmelde-Fenster

Gebe hier den User ``admin@admin.net`` mit dem Passwort ``Muster!`` ein und klicke auf ``Login``.

Nach erfolgreicher Anmeldung wirst Du darauf hingewiesen, dass Du XOA ``from Sources`` nutzt und Du daher kein Support und keine Updates erhälst.

.. figure:: media/60_xoa_login-from-sources.png
   :align: center
   :alt: XOA Hinweis aus den Quellen 

Bestätige dies, indem Du ``Ok`` klickst.

Danach siehst Du das ``Welcome-Fenster``. 

.. figure:: media/61_xoa_vm-first-screen.png
   :align: center
   :alt: XOA Willkommen Bildschirm

Du must nun den XCP-ng Host oder den XCP-ng Pool angeben, damit XOA hierauf zugreifen und die Ressourcen verwalten kann.
Wähle den Eintrag ``Add Server``.

Es erscheint dann das Einstellungs-Fenster für die Server (Settings).

.. figure:: media/62_xoa_add-xcp-ng-host.png
   :align: center
   :alt: Hinzufügen des XCP-ng-Hosts

Trage den Hostnamen, die IP-Adresse ``10.X.X.X`` ein, die Du dem XCP-ng Server gegeben hast und gebe dahinter - durch einen Doppelpunkt getrennt - den Port an.
I.d.R. ist dies Port 443, der zu nutzen ist. XCP-ng nutzt hierbei self-signed certificates. Trage den Benutzernamen des root-Benutzers von XCP-ng sowie sein Kennwort ein.
Setze zudem den Schiebeschalter nach rechts - auf grün -, damit nicht authorisierte Zertifikate - also self-signed certificates - akzeptiert werden.
Klicke auf ``Connect``. Es wird nun von der XOA-VM die Verbindung zum XCP-ng Host aufgebaut und gespeichert.

.. note::
   Falls Du einen XCP-ng Pool mit mehreren Servern und Speicherressourcen definiert hast, must Du hier nur den Pool-Master als Server eintragen. 
   Alle weiteren Server und Ressourcen werden dann automatisch erkannt.

Ändere nun das voreingestellte Kennwort für den root-Benutzer (admin@admin.net) der XOA-VM. Klicke hierzu auf der linken Menüleist ganz unten auf der Personensymbol.

.. figure:: media/63_xoa_edit-my-settings.png
   :align: center
   :alt: Benutzer-Symbol

Danach bist du im Kontexmenü des Benutzers, für den Du das Kennwort ändern und weitere Einstellungen vornehmen kannst.

.. figure:: media/64_xoa_edit-password.png
   :align: center
   :alt: Password Ämderung

Trage das bisherige Kennwort ``Muster!`` sowie zweimal Dein neunes Kennwort ein, stelle die Sprache ein und bestätige die Änderungen mit einem Klick auf ``OK``.

SSH-Verbindung zur VM
~~~~~~~~~~~~~~~~~~~~~

Um sich erstmalig mit der XOA-VM via SSH zu verbinden, gibst Du in einem Terminal ein:

.. code::

   ssh -p 22 muster@10.16.1.4

Bestätige den fingerprint mit ``yes``und gebe das Kennwort ``Muster!`` ein.

Gebe auf der Konsole ``passwd`` ein und ändere der Kennwort für den Benutzer ``muser``.

Wechsle auf der Konsole zum root-Benutzer, indem Du als Benutzer ``muster`` den Befehl ``sudo su`` angibst.
Du wirst nach dem Kennwort des Muster-Nutzers gefragt. Gebe das vorher geänderte Kennwort an. Du kannst nun als Benutzer ``root`` arbeiten.

Im Verzeichnis ``/root`` findet sich eine README-Datei mit Hinweisen zur VM sowie weitere Skripte zur Aktualisierung der XOA-Installation.

Update der XOA-Installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Um die XOA-Installation zu aktualisieren, findest Du ein Skript, das Du als root-Benutzer ausführen must.

Rufe das Skript ``/root/xo-update.sh`` auf. Die XOA-Installation from Sources wird aktualisiert. Hierbei wird aber die von linuxmuster.net angepasste
Konfigurationsdatei des xo-servers wieder überschrieben. Daher must Du nach dem Update noch die angepasste Konfigurationsdatei des xo-servers wieder zurückspielen. 
Diese Datei liegt unter ``/root/config.toml.backup`` und sollte dort niemals gelöscht werden!
Für die Rücksicherung der Konfigurationsdatei findest Du unter ``root/restore-xo-config.sh`` ein Skript, das Du als Benutzer ``root`` ausführen must. Die angepasste 
Konfigurationsdatei wird so an den korrekten Ort zurückgeschrieben und danach wir der xo-server neu gestartet.

Weitere Hinweise findest Du unter ``root/README``.

Backups: Backup NG
~~~~~~~~~~~~~~~~~~

Um mithilfe von XOA Backups zu definieren, wählst Du in der GUI der XOA-VM links im Menü den Eintrag ``Backup NG``. Dies ist der Eintrag, um Backups für XCP-ng zu erstellen.
Der Menüeintrag ``Backup`` existiert aufgrund der Abwärtskompatibilität zu XenServer -Installationen.

Grundlegende Erläuterungen zu den verschiedenen Backup-Möglichkeiten_ mit XOA findest Du im Handbuch zu XOA. Hier gibt es ebenfalls Einführungsvideos.

.. _Backup-Möglichkeiten: https://xen-orchestra.com/docs/backups.html

Wurden Backups definiert und wurden diese bereits ausgeführt, dann kannst Du deren Status und ggf. zusätzliche Backupinformationen aufrufen.

Dies kann dann z.B. wie in folgender Abbildung aussehen:

.. figure:: media/65_xoa_backup-ng.png
   :align: center
   :alt: XOA Backup

