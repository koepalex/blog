---
title: "Ubuntu und ZFS, Problem nach Update"
date: "2016-02-24"
categories: 
  - "ubuntu"
  - "zfs"
---

ZFS ist ein tolles, modernes Dateisystem, welches sich wachsender Beliebtheit erfreut. Leider ist es noch nicht direkt im Ubuntu enthalten, sondern wird als APT-Paket (ubuntu-zfs) nachinstalliert. Eine Anleitung dafür findet sich auf [ubuntuusers.de](https://wiki.ubuntuusers.de/ZFS_on_Linux/).

Nach einem Update hat oft zfs nicht mehr funktioniert. Es kam zur Fehlermeldung, dass zfs nicht gefunden werden konnte und man bitte mittels

modprobe zfs

Das entsprechende Kernelmodule laden soll. Der Befehl schlug dann auch immer Fehl mit einer Fehlermeldung wie "zfs.ko not found".

Wenn man nach diesem Problem sucht findet man einige Webseiten, die seit Ubuntu 12.04 von diesem Problem berichten. Und einige der empfohlenen _Lösungen_ haben bei mir auch funktioniert (bis zum nächsten Update). Seit gestern ist jetzt das Problem dauerhaft gelöst!

Des Rätzelslösung ist es das Paket **ubuntu-headers-generic** zu installieren, nur mit den Kernel-Headers können die Kernelmodule (zfs, spl) gebaut werden und damit dann auch verwendet werden.

Wieso es zu dem Problem kommt, ist etwas unverständlich. Ubuntu-zfs hat das APT-Paket **linux-headers** als empfohlenes Package verlinkt. APT installiert die empfohlenen Pakete automatisch mit, womit die Headers der installierten Kernel immer dabei sein sollten... Jedoch funktionierte das in mehreren Fällen nicht.
