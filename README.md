# ![](https://www.lpice.eu/fileadmin/_processed_/csm_LPIC-DevOpsToolsEngineer_43de3c4735.jpg) LPI DevOps Tools Engineer - Exam 701 & LPI Linux Essentials - Exam 010

Dokumention über den Lern- und Entwicklungsprozesses mit Ausgesuchten Unterkapiteln aus dem LPI DevOps Tools Engineer - Exam 701 & LPI Linux Essentials - Exam 010 des Wahlmoduls W901 an der TBZ.

***

# Inhaltsverzeichnis

<details>
<summary>Links</summary>
<br>

* [Exam 701: DevOps Tools Engineer](https://www.lpi.org/our-certifications/exam-701-objectives)
* [Linux Essentials Exam 010](https://www.lpi.org/our-certifications/exam-010-objectives)
* [GitHub - Exam 701: DevOps Tools Engineer](https://github.com/w901-fr19-mi/E701)
* [GitHub - E010 - Linux Essentials Exam 010](https://github.com/w901-fr19-mi/E010)
* [BSCW - W901 - Grundlagen Linux, UNIX Betriebsystem](https://bscw.tbz.ch/bscw/bscw.cgi/26211645)
* [Google Cloud Platform](https://console.cloud.google.com/)  
* [GitHub - Forked E701 Documentation](https://github.com/w901-fr19-mi/myE701)

<br>
</details>


<details>
<summary>Fahrplan</summary>
<br>

| Datum | behandelte Unterrichtsinhalte: | Gewichtung |
| -------- | ------ | -------- |
| 15.05.19 | Installation SW, Einrichten Linux VM(s)<br>[701.1 Modern Software Development, 1. Teil](https://github.com/w901-fr19-mi/E701#7011-modern-software-development) | 6 |
| 22.05.19 | [701.1 Modern Software Development, 2. Teil](https://github.com/w901-fr19-mi/E701#7011-modern-software-development) | 4 |
| 29.05.19 | [701.3 Source Code Management](https://github.com/w901-fr19-mi/E701#7013-source-code-management) | 5 | 
| 05.06.19 | 702.1 Container Usage, 1. Teil | 7 |
| 12.06.19 | 702.1 Container Usage, 2. Teil | (7) |
| 19.06.19 | 702.2 Container Deployment and Orchestration | 5 |
| 26.06.19 | LB1 Theoretische Prüfung und Abschluss LB2 | - |
| 03.07.19 | Sommersporttage | - |
|          | Total Punkte | 27 (34) !

<br>
</details>


<details>
<summary>Dokumentation des Lern- und Entwicklungsprozesses</summary>
<br>

<details>
<summary>Prerequisites</summary>
<br>

# Kubernetes Cluster
**Einrichten der Kubernetes Umgebung mit einem Master und zwei Worker Servern auf einem ESXi Host**

## Step 1 - Kubeadm installation (On all VMs)

### VM Setup
Zuerst müssen drei VMs erstellt und mit Ubuntu Server 18.04.02 LTS konfiguriert bzw. installiert werden.<br>
Nachdem die VMs erstellt, geupdatet und mit den richtigen Hostnames & IP-Adressen konfiguriert wurde, muss noch das Hostsfile angepasst werden.<br>


    sudo nano /etc/hosts

Folgende konfiguration muss bei allen drei Servern ins Hostfile geschrieben werden.

    10.10.5.100 s801-k8sm-01
    10.10.5.110 s802-kwrk-01
    10.10.5.111 s803-kwrk-02

Schlussendlich muss bzw. kann man die Konfiguration auf allen drei Server mit folgenden Befehlen überprüfen.

    ping -c 3 s801-k8sm-01
    ping -c 3 s802-kwrk-01
    ping -c 3 s803-kwrk-02

Alle drei VMs sollten nun die Hostnamen zur IP-Adresse auflösen können. 

### Docker installation

Mit folgendem Befehl Docker auf den drei VMs installieren. Diese Docker Version ist die bereits compilierte vom Ubuntu Repository<br>

    sudo apt install docker.io -y

Danach muss man noch den Docker Service aktivieren und einstellen, dass er bei jedem Systemstart mit startet.<br>

    sudo systemctl start docker
    sudo systemctl enable docker

### SWAP deaktivieren

SWAP Partition finden und temporär deaktivieren.<br>

    sudo swapon -s
    sudo swapoff -a

SWAP permanent deaktivieren.<br>

    sudo nano /etc/fstab

Sobald die Konfigurationdatei offen ist, die zuvor gefundenen SWAP Partition mit einem # auskommentieren, die Konfigurationdatei wieder abspeichern und die VM neustarten.

    sudo init 6

### Kubeadm packete installieren



</details>

<details>
<summary>LPI DevOps Tools Engineer - Exam 701</summary>
<br>

# Kapitel: 702.1 Container Usage (Status: In Arbeit)

**Weight**: 7 (7)

**Beschreibung** Gegenüberstellung welche Linux Technologien für Container verwendet werden.

**Tagesziele**, z.B. Erstellung einer Tabelle Linux - Container. 

**Vorgehen**, z.B. Studieren Background Linux Namespaces vs. Container, UnionFS vs. Container Layer, Unix Prozesse (Jobs) vs. Docker run/start/stop

**Beispiele und Arbeitsergebnisse**

| Linux          | Container      | Beschreibung      |
| -------------- | -------------- | ----------------- |
| Namespaces     | laufender Container | beim Starten des Containers wird in eine andere Linux Namespace gewechselt |
| UnionFS        | Image Layer         | Container Verwenden UnionFileSysteme um .... |
| Unix Prozesse  | run/start/stop      | docker run/start/stop Befehle ähneln dem .... Subsystem |

**Fazit und Aussicht**, z.B. Die Durcharbeitung von ... gab mir ein besseres Verständnis über die Funktionsweise von Containern.

***

</details>


<details>
<summary>LPI Linux Essentials - Exam 010</summary>
<br>



</details>

</details>