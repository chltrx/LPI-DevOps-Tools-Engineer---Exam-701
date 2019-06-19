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
| 15.05.19 | 701.1 Modern Software Development | 6 |
| 22.05.19 | 701.2 Standard Components and Platforms for Software | 2 |
| 29.05.19 | Kubernetes Cluster installieren & konfigurieren | 14 | 
| 05.06.19 | 701.3 Source Code Management | 5 |
| 12.06.19 | 702.1 Container Usage | 7 |
| 19.06.19 | 702.2 Container Deployment and Orchestration | 5 |
| 26.06.19 | LB1 Theoretische Prüfung und Abschluss LB2 | - |
| 03.07.19 | Sommersporttage | - |
|          | Total Punkte | 39

<br>
</details>


<details>
<summary>Dokumentation des Lern- und Entwicklungsprozesses</summary>
<br>

<details>
<summary>Kubernetes Cluster installieren & konfigurieren</summary>
<br>

# Kubernetes Cluster

**Weight**: 14 

**Beschreibung**, Einrichten der Kubernetes Umgebung mit einem Master und zwei Worker Servern auf einem ESXi Host

**Tagesziele**, Laufende Kubernetes Umgebung mit einem Master und zwei Worker Servern.

**Vorgehen**, Informationen über Kubernetes und Kubernetes Cluster unter Linux bzw. Ubuntu suchen. Verschiedene Anleitungen studieren und schlussendlich das Kubernetes Cluster aufbauen.

**Beispiele und Arbeitsergebnisse**

## Step 1 - Kubeadm installation
Betroffene VMs<br>

    s801-k8sm-01 <- Master
    s802-kwrk-01 <- Worker 1
    s803-kwrk-02 <- Worker 2

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

Dependencies installieren<br>

    sudo apt install -y apt-transport-https

Kubernetes Key hinzufügen<br>

    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

Kubernetes Repository hinzufügen.<br>

    cd /etc/apt/
    sudo nano sources.list.d/kubernetes.list

Kuberentes Repository in die zuvor erstellte Liste einfügen.<br>

    deb http://apt.kubernetes.io/ kubernetes-xenial main

Repositories Updaten und "kubeadm", "kubelet" und "kubectl" installieren.<br>

    sudo apt update
    sudo apt install -y kubeadm kubelet kubectl

## Step 2 - Kubernetes Cluster initialisieren
Betroffene VMs:<br>

    s801-k8sm-01 <- Master
### Cluster initialisieren

    sudo kubeadm init --pod-network-cidr=10.13.37.0/24 --apiserver-advertise-address=10.10.5.100 --kubernetes-version "1.14.2"

Den kubeadm join Command kopieren, dieser wird später benötigt um die Worker ins Cluster ein zu binden.<br>

#### Note

| Parameter          | Description      |
| -------------- | -------------- |
| --apiserver-advertise-address     | Determines which IP address Kubernetes should advertise its API server on. |
| --pod-network-cidr        | Specify the range of IP addresses for the pod network. We're using the 'flannel' virtual network. If you want to use another pod network such as weave-net or calico, change the range IP address.         |

### Create .kube Configuration

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

### Deploy flannel network

    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

### Check kubernetes node & pads

    kubectl get nodes
    kubectl get pods --all-namespaces

## Step 3 - Adding Worker Nodes to the Kubernetes Cluster
Betroffene VMs<br>

    s802-kwrk-01 <- Worker 1
    s803-kwrk-02 <- Worker 2

### Join Worker 1 & 2

    kubeadm join 10.10.5.100:6443 --token p63z79.ihri5v5hdskjbhdj --discovery-token-ca-cert-hash sha256:11d78c9595bdbe95995517d96ce17b93adc1681e2086483a760a9d99f16f2edd

### Test
Betroffene VMs<br>

    s801-k8sm-01 <- Master

#### Testen ob die Worker erfolgreich gejoint wurden

    kubectl get nodes

# Kubernetes Dashboard UI
**Installieren des Kubernetes Dashboard UIs um die Container via Web-Interface zu verwalten.**
Betroffene VMs<br>

    s801-k8sm-01 <- Master

## Dashboard UI installation

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploay/recommended/kubernetes-dashboard.yaml

## Activate Dashboard UI

    kubectl proxy

## Access Dashboard UI

    http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

### Note

The UI can only be accessed from the machine where the command is executed. See kubectl proxy --help for more options.

**Fazit und Aussicht**, Die erstellung des Kubernetes Clusters gab mir ein besseres Verständnis darüber wie die Infrastruktur hinter einem Cluster funktioniert, wie die einzelnen Server bzw. Master und Worker kommunizieren und wie das ganze aufgesetzt/erstellt werden kann.

</details>

<details>
<summary>LPI DevOps Tools Engineer - Exam 701</summary>
<br>

# Kapitel: 701.1 Modern Software Development (Status: Abgeschlossen)

**Weight**: 6 + 4 Bonsupunkte für die Erstellung eines Microservices inkl Unit-Tests.

**Beschreibung**, Gegenüberstellung wie ein Microservice ... handhabt:
* Data persistence
* Sessions
* Status information
* Transactions
* Concurrency
* Security
* Performance
* Availablility
* Scaling
* Load balancing
* Messaging
* Monitoring
* APIs

**Tagesziele**, Aufstellung der oben genannten Stichworte. 

**Vorgehen**, Informationen über die oben gennanten Stichworte sammeln, den bezug zu Microservices herausfinden und schlussendlich in einer Aufstellung beschreiben.

**Beispiele und Arbeitsergebnisse**

## Data persistence
### Database Per Service
In diesem Muster verwaltet jeder Mikrodienst seine eigenen Daten. Dies impliziert, dass kein anderer Mikrodienst direkt auf diese Daten zugreifen kann. Die Kommunikation oder der Datenaustausch kann nur über eine Reihe gut definierter APIs erfolgen.

### Shared Database
Eine gemeinsam genutzte Datenbank kann eine sinnvolle Option sein, wenn die Herausforderungen im Zusammenhang mit Database Per Service für Ihr Team zu schwierig werden.
Dieser Ansatz versucht, die gleichen Probleme zu lösen. Dies geschieht jedoch durch einen wesentlich nachgiebigeren Ansatz, indem eine gemeinsam genutzte Datenbank verwendet wird, auf die mehrere Mikrodienste zugreifen.
Meistens ist dies ein sichereres Muster für Entwickler, da sie in der Lage sind,auf bestehende Weise zu arbeiten. Bekannte ACID-Transaktionen werden verwendet,um die Konsistenz zu gewährleisten.
Durch diesen Ansatz werden jedoch die meisten Vorteile von Microservices zunichte gemacht. Teamübergreifende Entwickler müssen die Schemaänderungen an Tabellen koordinieren. Es kann auch zu Laufzeitkonflikten kommen, wenn mehrere Dienste versuchen, auf dieselben Datenbankressourcen zuzugreifen.
Insgesamt kann dieser Ansatz langfristig mehr schaden als nützen.

### Saga Pattern
Das Saga-Muster ist die Lösung für die Implementierung von Geschäftstransaktionen, die sich über mehrere Microservices erstrecken.
Eine  Saga  ist im Grunde eine Folge lokaler Transaktionen. Für jede innerhalb einer Saga durchgeführte Transaktion veröffentlicht der Dienst, der die Transaktion durchführt, ein Ereignis. Die nachfolgende Transaktion wird basierend auf der Ausgabe der vorherigen Transaktion ausgelöst. Und wenn eine der Transaktionen in dieser Kette fehlschlägt, führt die Saga eine Reihe von Ausgleichstransaktionen aus, um die Auswirkungen aller vorherigen Transaktionen rückgängig zu machen.

### API Composition
Dieses Muster ist eine direkte Lösung für das Problem der Implementierung komplexer Abfragen in einer Microservices-Architektur.
In diesem Muster ruft ein API Composer andere Microservices in der erforderlichen Reihenfolge auf. Nach dem Abrufen der Ergebnisse werden die Daten im Arbeitsspeicher verknüpft, bevor sie dem Verbraucher zur Verfügung gestellt werden.
Wie offensichtlich ist der Nachteil dieses Musters die Verwendung ineffizienter speicherinterner Verknüpfungen für potenziell große Datasets.

### CQRS - Command Query Responsibility Segragation
CQRS (Command Query Responsibility Segregation) ist ein Versuch, die Probleme mit dem API Composition Pattern zu umgehen.
Eine Anwendung überwacht Domänenereignisse von anderen Microservices und aktualisiert die Ansicht oder die Abfragedatenbank. Sie können komplexe Aggregationsabfragen aus dieser Datenbank bedienen. Sie können die Leistung optimieren und die Abfrage-Microservices entsprechend skalieren.
Der Nachteil dabei ist eine zunehmende Komplexität. Plötzlich sollte Ihr Microservice Ereignisse verarbeiten. Dies kann zu Latenzproblemen führen, bei denen die Ansichtsdatenbank möglicherweise konsistent ist und nicht immer konsistent. Es kann auch die Codeduplizierung erhöhen.

### Event Sourcing
Die Ereignisbeschaffung versucht hauptsächlich, das Problem der atomaren Aktualisierung der Datenbank und der Veröffentlichung eines Ereignisses zu lösen.
Bei der Ereignisbeschaffung speichern Sie den Status der Entität oder des Aggregats als Folge von Statusänderungsereignissen. Ein neues Ereignis wird jedes Mal erstellt, wenn ein Update oder eine Einfügung vorliegt. Der Ereignisspeicher wird zum Speichern der Ereignisse verwendet.


## Sessions
### Sticky sessions
Dadurch wird sichergestellt, dass alle Anforderungen eines bestimmten Benutzers an denselben Server gesendet werden, der die erste Anforderung für diesen Benutzer bearbeitet hat. Auf diese Weise wird sichergestellt, dass die Sitzungsdaten für einen bestimmten Benutzer immer korrekt sind. Diese Lösung hängt jedoch vom Load Balancer ab und kann nur das horizontal erweiterte Cluster-Szenario erfüllen. Wenn der Load Balancer jedoch aus irgendeinem Grund plötzlich gezwungen wird, Benutzer auf einen anderen Server zu verschieben, gehen alle Sitzungsdaten des Benutzers verloren.

### Session replication
Bedeutet, dass jede Instanz alle Sitzungsdaten speichert und über das Netzwerk synchronisiert. Das Synchronisieren von Sitzungsdaten verursacht einen Mehraufwand für die Netzwerkbandbreite. Solange sich die Sitzungsdaten ändern, müssen die Daten mit allen anderen Computern synchronisiert werden. Je mehr Instanzen, desto mehr Netzwerkbandbreite bringt die Synchronisation.

### Centralized session storage
Bedeutet, dass beim Zugriff eines Benutzers auf einen Mikrodienst Benutzerdaten aus dem gemeinsamen Sitzungsspeicher abgerufen werden können, um sicherzustellen, dass alle Mikrodienste dieselben Sitzungsdaten lesen können. In einigen Szenarien ist dieses Schema sehr gut und der Anmeldestatus des Benutzers ist undurchsichtig. Es ist auch eine hochverfügbare und skalierbare Lösung. Der Nachteil dieser Lösung ist jedoch, dass der gemeinsame Sitzungsspeicher einen bestimmten Schutzmechanismus erfordert und daher auf sichere Weise zugänglich sein muss.


## Status information / Monitoring
### Application Metrics
Diese Metriken beziehen sich speziell auf Ihre Anwendung. Wenn Ihre Anwendung Benutzerregistrierungen akzeptiert, kann eine Standardmetrik sein, wie viele in der letzten Stunde erfolgreich abgeschlossen wurden. Ein Steuervorbereitungssystem kann kontextspezifische Ereignisse wie die Formularvalidierung aufzeichnen. Diese Daten auf oberster Ebene sind für Entwicklungsteams und die Organisation hilfreich, um das Funktionsverhalten des Systems zu verstehen. Wenn die Validierung von Formularen mit maximalem Volumen in der Regel 1.000 Mal pro Stunde erfolgt und dieser Durchsatz in den letzten zwei Stunden plötzlich auf 500 sinkt, kann diese Anomalie ein Hinweis auf ein Problem sein.

### Platform Metircs
Diese Metriken geben Aufschluss über die Hauptmerkmale Ihrer Infrastruktur. Beispiele beinhalten:
Die durchschnittliche Gesamtausführungszeit für jede der zehn am häufigsten ausgeführten Datenbankabfragen.
Die durchschnittliche Ausführungszeit der schnellsten 10 Prozent;
Die durchschnittliche Ausführungszeit der langsamsten 10 Prozent.
Die Anzahl der Anfragen pro Minute, die ein Dienst erhält.
Die durchschnittliche Antwortzeit für jeden Service-Endpunkt.
Das Erfolgs / Fehler-Verhältnis für jeden Dienst.
Zusammen bieten diese Metriken ein Dashboard, mit dem die Leistung und das Verhalten von Systemen auf niedriger Ebene erfasst werden können. Im Idealfall werden Sie durch diese Metriken auf Leistungseinbußen aufmerksam gemacht, die sich wahrscheinlich auf den Gesamtdurchsatz auswirken oder zu einem systemweiten Ausfall führen. Es ist wichtig, Low-Level-Daten auf diese Weise zu verwenden, um größere Fehler vorherzusagen und zu verhindern, bevor sie auftreten.
Die Raygun APM-Plattform bietet ein Dashboard, mit dem diese und andere Arten von Metriken in Bezug auf die Gesamtsystemleistung visualisiert werden können.

### System Events
Äußere Kräfte wirken ständig auf Systeme. Am häufigsten und möglicherweise am störendsten sind neue Bereitstellungen. Das Betriebspersonal weiß, dass eine starke Korrelation zwischen neuen Code-Bereitstellungen und Systemfehlern besteht. Angesichts dieser Realität ist es sinnvoll, ein Protokoll dieser Bereitstellungen in die Zeitreihendaten aufzunehmen, die von den Anwendungen zusammen mit den übrigen Metriken aufgezeichnet werden. Skalierungsereignisse, Konfigurationsaktualisierungen und andere betriebliche Änderungen sind ebenfalls relevant und sollten aufgezeichnet werden. Das Aufzeichnen dieser Ereignisse ermöglicht es auch, sie mit dem Systemverhalten zu korrelieren.

### Überwachungs-Tools
**Raygun APM** : Die APM-Plattform von Raygun ist ein weiteres Beispiel für ein komplettes System, das sowohl Instrumentierungs- als auch Erfassungsprozesse sowie ein Dashboard zur Visualisierung von Messdaten bietet. Raygun APM unterstützt .NET. Java- und Ruby-Unterstützung sind in Entwicklung.<br>

**Zipkin** : Zipkin ist ein Open-Source-Verfolgungssystem, das speziell für die Verfolgung von Anrufen zwischen Mikrodiensten entwickelt wurde. Es ist besonders nützlich für die Analyse von Latenzproblemen. Zipkin enthält sowohl Instrumentenbibliotheken als auch die Collector-Prozesse, die Ablaufverfolgungsdaten erfassen und speichern.<br>

**Apache Kafka** : Kafka ist ein Streams-Verarbeitungssystem. Es verwendet eine Publish / Subscribe-Methode zum Lesen und Schreiben von Daten in einen logischen Stream, dessen Konzept einem Messaging-System wie RabbitMQ ähnelt. Kafka kann mit anderen Tools wie Zipkin kombiniert werden, um eine robuste Lösung für die Übertragung und Speicherung von Messdaten bereitzustellen.<br>

**Grafana** : Die mit all diesen Tools gesammelten Daten sind nur dann sehr nützlich, wenn sie interpretiert und analysiert werden können. Zu diesem Zweck ist Grafana ein webbasiertes Visualisierungstool, mit dem visuelle Dashboards erstellt werden.<br>

**Prometheus** : Prometheus ist eine Open-Source-Überwachungslösung, die ursprünglich von SoundCloud entwickelt wurde. Es wird häufig zum Speichern und Abfragen von „Zeitreihendaten“ verwendet. Hierbei handelt es sich um Daten, die Aktionen im Zeitverlauf beschreiben. Prometheus wird häufig mit anderen Tools, insbesondere Grafana, kombiniert, um die Zeitreihendaten zu visualisieren und Dashboards bereitzustellen.


## Transactions
Ohne Transaktionen können keine Echtzeitanwendung verwendet werden kann. Bei der Transaktion kann es sich um Geldtransfer, E-Mail-Versand, Auftragserteilung,Rechnungszahlung usw. handeln. Vereinfacht ausgedrückt, handelt es sich bei einer Transaktion um eine Arbeitseinheit, die in jeder Geschäftsanwendung ausgeführt wird, in der Daten in einem bestimmten Zustand erhalten bleiben, jedoch um jede Einheit von Die Arbeit sollte vollständig festgeschrieben oder rückgängig gemacht werden, um die Datenintegrität sicherzustellen.

### Monolithic vs. Microservices
Monolithische Anwendungen bleiben beim Umgang mit Transaktionen in erster Linie bei den ACID-Eigenschaften. Der größte Vorteil für die monolithische Anwendung im Transaktionsmanagement ist ein einziger und gemeinsamer Datenbankserver. Transaktionen können auf Datenbankebene initiiert und basierend auf dem Endergebnis der Transaktion festgeschrieben oder rückgängig gemacht werden. Diese Art des Initiierens einer Transaktion, Durchführen von Vorgängen und Beibehalten oder Nichtbeibehalten von Daten ist viel einfacher zu entwickeln. Dieser Vorteil wird jedoch zu einer ernsthaften Herausforderung bei der Ausführung von Großserienanwendungen. Eine einfache Sperre einer kritischen Tabelle kann zu katastrophalen Ergebnissen führen, z. B. zur Nichtverfügbarkeit der gesamten Anwendung.
In den Richtlinien von Microservices wird dringend empfohlen, das Single Repository Principle (SRP) zu verwenden. Dies bedeutet, dass jeder Mikrodienst seine eigene Datenbank verwaltet und kein anderer Dienst direkt auf die Datenbank des anderen Dienstes zugreifen sollte. Es gibt keine direkte und einfache Möglichkeit, ACID-Prinzipien über mehrere Datenbanken hinweg zu verwalten. Auf diese Weise liegt die eigentliche Herausforderung für das Transaktionsmanagement in Microservices.


## Concurrency
In Microservices kann eine logisch atomare Operation häufig mehrere Microservices umfassen. Sogar ein monolithisches System kann mehrere Datenbanken oder Messaging-Lösungen verwenden. Bei mehreren unabhängigen Datenspeicherungslösungen besteht das Risiko inkonsistenter Daten, wenn einer der verteilten Prozessteilnehmer ausfällt, z. B. wenn ein Kunde belastet wird, ohne eine Bestellung aufzugeben, oder der Kunde nicht benachrichtigt wird, dass die Bestellung erfolgreich war.<br>

Solange wir mehrere Speicherorte für die Daten haben (die sich nicht in einer einzigen Datenbank befinden), wird die Konsistenz nicht automatisch gelöst, und die Ingenieure müssen sich beim Entwerfen des Systems um die Konsistenz kümmern. Momentan gibt es meiner Meinung nach in der Branche noch keine allgemein bekannte Lösung für die atomare Aktualisierung von Daten in mehreren verschiedenen Datenquellen - und wir sollten wahrscheinlich nicht abwarten, bis eine verfügbar ist.

Ein Versuch, dieses Problem auf automatisierte und problemlose Weise zu lösen, ist das XA-Protokoll, das das Zwei-Phasen-Festschreibungsmuster (2PC) implementiert . In modernen High-Scale-Anwendungen (insbesondere in einer Cloud-Umgebung) scheint 2PC jedoch nicht so gut zu funktionieren. Um die Nachteile von 2PC zu beseitigen, müssen wir ACID gegen BASE eintauschen und uns je nach Anforderung auf unterschiedliche Weise um die Konsistenz kümmern.<br>

**Saga-Muster**
Die bekannteste Methode zur Behandlung von Konsistenzproblemen bei mehreren Mikrodiensten ist das Saga-Muster. Sie können Sagas als verteilte Koordination mehrerer Transaktionen auf Anwendungsebene behandeln. Je nach Anwendungsfall und Anforderungen optimieren Sie Ihre eigene Saga-Implementierung. Im Gegensatz dazu versucht das XA-Protokoll, alle Szenarien abzudecken. Das Saga-Muster ist auch nicht neu. Es war in der Vergangenheit bekannt und wurde in ESB- und SOA-Architekturen verwendet. Schließlich gelang der Übergang in die Welt der Mikrodienstleistungen. Jeder atomare Geschäftsvorgang, der mehrere Services umfasst, kann aus mehreren Transaktionen auf technischer Ebene bestehen. Die Schlüsselidee des Saga-Musters besteht darin, eine der einzelnen Transaktionen rückgängig machen zu können. Wie wir wissen, ist ein Rollback für bereits festgeschriebene Einzeltransaktionen nicht möglich. Dies wird jedoch durch Aufrufen einer Kompensationsaktion erreicht - durch Einführen einer "Abbrechen" -Operation.


## Security
### OAuth für die Benutzeridentitäts und Zugriffskontrolle
Die überwiegende Mehrheit der Anwendungen muss ein gewisses Maß an Zugriffskontrolle und Berechtigungsverwaltung durchführen . Was Sie hier vermeiden möchten, ist das Rad neu zu erfinden. OAuth / OAuth2 ist hinsichtlich der Benutzerautorisierung praktisch der Industriestandard. Obwohl das Erstellen eines eigenen benutzerdefinierten Autorisierungsprotokolls eine Option ist, wird es von vielen Anbietern nicht empfohlen, es sei denn, Sie haben starke und sehr spezifische Gründe dafür.
Während OAuth2  nicht perfekt ist , ist es ein weit verbreiteter Standard. Der Vorteil der Verwendung besteht darin, dass Sie sich auf Bibliotheken und Plattformen verlassen können, die Ihre Entwicklungsphase erheblich beschleunigen. Aus dem gleichen Grund haben bereits einige der größten Unternehmen und intelligentesten Ingenieure verschiedene Lösungen zur Verbesserung des Sicherheitsniveaus Ihres OAuth-basierten Autorisierungsdienstes entwickelt.

### "Defence in depth" um Dienste zu priorisieren
Es ist ein großer Fehler, wenn Sie davon ausgehen, dass eine Firewall in Ihrem Netzwerk zum Schutz Ihrer Software ausreicht. "Defence in depth" ist definiert als "ein Informationssicherungskonzept, bei dem mehrere Ebenen von Sicherheitskontrollen (Verteidigung) in einem Informationstechnologiesystem angeordnet sind."

Im Klartext müssen Sie lediglich die vertraulichsten Dienste identifizieren und ihnen eine Reihe von verschiedenen Sicherheitsebenen zuweisen, damit ein potenzieller Angreifer, der eine Ihrer Sicherheitsebenen ausnutzen kann, diese noch ermitteln muss Sie haben die Möglichkeit, alle anderen Abwehrmechanismen bei Ihren kritischen Services zu überwinden.

### Keine eigenen Krypto-Codes schreiben
Im Laufe der Jahre haben viele Menschen unglaublich viel Geld, Zeit und Ressourcen in den Bau von Bibliotheken investiert, die sich mit Ver- und Entschlüsselung befassen. Wenn Sie 10 intelligente und kompetente Sicherheitsleute anheuern, sie alle in einen Raum stellen und sie bitten würden, die bestmögliche Bibliothek für die Ver- und Entschlüsselung zu entwickeln, würden sie zweifellos so gut sein wie die besten Open-Source-Kryptobibliotheken, die es gibt sind schon da draußen.
Wenn es um Sicherheit geht, sollten Sie die meiste Zeit nicht versuchen, Ihre eigenen neuen Lösungen und Algorithmen zu entwickeln, es sei denn, Sie haben starke und spezifische Gründe dafür und Sie haben Leute, die qualifiziert genug sind, um etwas zu schaffen, das beinahe so gut ist wie das Open-Source-Tools sind bereits verfügbar (Tools, die von der Community intensiv getestet wurden).
In den meisten Fällen sollten Sie  NaCl / libsodium  zur Verschlüsselung verwenden. Es gibt es schon seit mehreren Jahren und es ist schnell, einfach zu bedienen und sicher. Die ursprüngliche Implementierung von NaCl ist in C geschrieben , unterstützt aber auch C ++ und Python . Und dank der libsodium-Fork sind verschiedene Adapter für andere Sprachen wie PHP , Javascript und Go  verfügbar.

### Automatische Sicherheitsupdates
Wenn Sie möchten, dass Ihre Microservices-Architektur gleichzeitig sicher und skalierbar ist, ist es eine gute Idee, in der frühen Entwicklungsphase einen Weg zu finden, um alle Ihre Software-Updates zu automatisieren oder zumindest unter Kontrolle zu halten.
Eine hohe Testabdeckung ist hier wichtiger denn je. Jedes Mal, wenn ein Teil Ihres Systems aktualisiert wird, möchten Sie sicherstellen, dass Sie Probleme früh genug und so detailliert wie möglich erkennen.
Stellen Sie sicher, dass Ihre Plattform größtenteils " atomar" ist. Das bedeutet, dass alles in Container eingeschlossen werden sollte,  sodass das Testen Ihrer Anwendung mit einer aktualisierten Bibliothek oder Sprachversion nur eine Frage des Umschließens eines anderen Containers ist. Sollte der Vorgang fehlschlagen, ist das Umkehren ziemlich einfach und kann vor allem automatisiert werden.

### Überwachung
Sie können es sich nicht leisten, ein verteiltes System ohne eine solide, fortschrittliche und zuverlässige Überwachungsplattform zu betreiben. Es gibt verschiedene Lösungen, aber die speziell für Microservices entwickelte Lösung, die es bisher gab, ist Prometheus.
Prometheus wurde ursprünglich von Ingenieuren bei SoundCloud entwickelt und ist eine Open-Source-Überwachungsplattform und Teil der Cloud Native Computing Foundation . Es wird von einigen der größten Akteure der Branche wie SoundCloud selbst, CoreOS und Digital Ocean unterstützt und übernommen.
Andere Überwachungslösungen umfassen InfluxDB , statsd und mehrere bekannte kommerzielle Plattformen.

### Sicherheitsscanner
Innerhalb Ihrer automatisierten Testsuite ist es sinnvoll, regelmäßige Schwachstellen- und Sicherheitsüberprüfungen für Ihre Container durchzuführen. Der Hauptdarsteller von Open Source in diesem Bereich scheint Clair von CoreOS zu sein. Docker Security Scanning und Twistlock sind einige kommerzielle Optionen.
Beachten Sie außerdem, dass das Container-Image selbst möglicherweise nur dann als vertrauenswürdig eingestuft wird, wenn seine Signatur überprüft wurde. rkt tut dies standardmäßig, während Docker vor einiger Zeit eine ähnliche Funktion einführte, nachdem mehrere Sicherheitslücken gefunden wurden.


## Performance
Effizienz ist von größter Bedeutung in der realen, groß angelegten Architektur verteilter Systeme, und Mikroservice-Ökosysteme bilden keine Ausnahme von dieser Regel. Es ist einfach, die Effizienz eines einzelnen Systems (wie bei einer monolithischen Anwendung) zu quantifizieren, aber die Bewertung der Effizienz und die Erzielung einer höheren Effizienz in einem großen Ökosystem von Mikrodiensten, in dem Aufgaben zwischen Hunderten (wenn nicht Tausenden) von kleinen Diensten aufgeteilt werden, ist unglaublich schwer. Es ist auch durch die Gesetze der Computerarchitektur und verteilter Systeme begrenzt, die die Effizienz großer, komplexer verteilter Systeme einschränken: Je verteilter Ihr System ist und je mehr Mikrodienste Sie in diesem System einsetzen, desto geringer ist die Wahrscheinlichkeit Unterschied wird die Effizienz eines Microservices auf dem gesamten System haben. Die Vereinheitlichung von Grundsätzen zur Steigerung der Gesamteffizienz wird zur Notwendigkeit.


## Availability
Ein Mikroservice muss ausfallsicher sein und häufig auf einem anderen Computer neu gestartet werden können, um die Verfügbarkeit zu gewährleisten. Diese Ausfallsicherheit hängt auch von dem Status ab, der für den Mikrodienst gespeichert wurde, von dem der Mikrodienst diesen Status wiederherstellen kann, und davon, ob der Mikrodienst erfolgreich neu gestartet werden kann. Mit anderen Worten, es muss eine Ausfallsicherheit in der Rechenkapazität (der Prozess kann jederzeit neu gestartet werden) sowie eine Ausfallsicherheit im Zustand oder in den Daten (kein Datenverlust, und die Daten bleiben konsistent) bestehen.Die Probleme der Ausfallsicherheit verschärfen sich in anderen Szenarien, z. B. wenn während eines Anwendungsupgrades Fehler auftreten. Der mit dem Bereitstellungssystem zusammenarbeitende Microservice muss bestimmen, ob weiterhin auf die neuere Version oder auf eine frühere Version zurückgegriffen werden kann, um einen konsistenten Status beizubehalten. Fragen, wie zum Beispiel, ob genügend Maschinen verfügbar sind, um weiter voranzukommen, und wie frühere Versionen des Microservices wiederhergestellt werden können, müssen berücksichtigt werden. Dazu muss der Mikrodienst Integritätsinformationen senden, damit die gesamte Anwendung und der Orchestrator diese Entscheidungen treffen können.


## Scaling
Der Skalierbarkeitswürfel beschreibt die Skalierbarkeit anhand eines Würfelmodells. Der "Skalierungswürfel" besteht aus einer X-Achse, einer Y-Achse und einer Z-Achse. Die traditionelle Methode zur Skalierung durch Ausführen mehrerer Kopien einer Anwendung, die auf mehrere Server verteilt ist, ist die X-Achse.<br>

Der allgemeine Ansatz von Microservices verläuft entlang der Y-Achse. Die Skalierung auf der Y-Achse unterteilt die Anwendung in ihre Komponenten und Dienste. Der Softwarearchitekt Chris Richardson erklärte diese Methode in einem Blogbeitrag: "Jeder Dienst ist für eine oder mehrere eng verwandte Funktionen verantwortlich. Es gibt verschiedene Möglichkeiten, die Anwendung in Dienste zu zerlegen. Ein Ansatz besteht darin, verbbasierte Zerlegung und Definition zu verwenden Services, die einen einzelnen Anwendungsfall implementieren, z. B. Checkout. Die andere Option besteht darin, die Anwendung nach Nomen zu zerlegen und Services zu erstellen, die für alle Vorgänge in Bezug auf eine bestimmte Entität verantwortlich sind, z -basierte Zersetzung."<br>

Das verlässt die Z-Achse. Die X-Achse ist eine traditionelle Skalierung des Lastausgleichs und die Y-Achse umfasst Microservices. Die Z-Achse verfolgt einen ähnlichen Ansatz wie die X-Achse: Sie führt identische Codekopien auf mehreren Servern aus. Was die Z-Achsen-Skalierung einzigartig macht, ist, dass sie auch eine Seite von der Y-Achse ausleiht, sodass jeder Server nur für eine Teilmenge der Anwendung und nicht für die gesamte Anwendung verantwortlich ist.<br>


## Load Balancing
### Server Side Load Blancing
In der Java EE-Architektur stellen wir unsere War- / Ear-Dateien auf mehreren Anwendungsservern bereit, erstellen dann einen Serverpool und stellen einen Load Balancer (Netscaler) davor, der eine öffentliche IP hat. Der Client stellt eine Anfrage unter Verwendung dieser öffentlichen IP, und Netscaler entscheidet, auf welchem ​​internen Anwendungsserver die Anfrage per Round-Robin- oder Sticky-Session-Algorithmus weitergeleitet wird. Wir nennen es Server Side Load Balancing.<br>

**Problem:**  Wenn ein oder mehrere Server nicht mehr reagieren, müssen Sie diese Server manuell aus dem Load Balancer entfernen, indem Sie die IP-Tabelle des Load Balancers aktualisieren.
Ein weiteres Problem besteht darin, dass wir eine Failover-Richtlinie implementieren müssen, um dem Client ein nahtloses Erlebnis zu bieten.
Microservices verwenden jedoch keinen serverseitigen Lastenausgleich. Sie verwenden clientseitigen Lastenausgleich.

### Client Side Load Balancing
Beim clientseitigen Lastausgleich wird ein Algorithmus wie Round-Robin oder zonenspezifisch beibehalten. über die es Instanzen von aufrufenden Diensten aufrufen kann. Der Vorteil ist, dass sich die Serviceregistrierung immer selbst aktualisiert. Wenn eine Instanz ausfällt, wird sie aus der Registrierung entfernt. Wenn der clientseitige Load Balancer mit dem Eureka-Server kommuniziert, aktualisiert er sich immer selbst, sodass im Gegensatz zum serverseitigen Load Balancing keine manuellen Eingriffe zum Entfernen einer Instanz erforderlich sind.
Ein weiterer Vorteil ist, dass Sie den Load-Balancing-Algorithmus programmgesteuert steuern können, da sich der Load-Balancer auf der Clientseite befindet. Ribbon bietet diese Möglichkeit, daher werden wir Ribbon für den clientseitigen Lastenausgleich verwenden.


## Messaging
Beim Übergang von einer monolithischen Anwendungsstruktur zu einer multi-unabhängigen Struktur stellen Sie sofort fest, dass das Aufrufen anderer Komponenten nicht auf ein System mit nur einem Prozess ausgerichtet ist. Innerhalb eines Single-Process-Systems rufen Sie normalerweise Komponenten mit verschiedenen Sprachmethoden oder über Dependency Injection auf, wie wir es bei Angular sehen. Da eine microservices-basierte Anwendung auf einer Vielzahl von Servern, Hosts und Prozessen ausgeführt werden kann, ist die Kommunikation auf HTTP (Hyper Text Transfer Protocol), TCP (Transmission Control Protocol) und AMQP (Advanced Message Queuing Protocol) ausgerichtet. Alle diese Protokolle sind für die IPC- oder Interprozesskommunikation konzipiert, da sie gemeinsam genutzte Daten verwalten. <br>

* Synchronous protocol
* Asynchronous protocol
* Single receiver
* Multiple receivers

### Synchronous Protocol
Synchronous Protocol ist in Chat-Funktionen, HTTP, Instant Messaging und Live-Funktionen integriert ist. Es handelt sich um eine Datenübertragung, die in regelmäßigen Abständen stattfindet und in der Regel von der Taktung des Mikroprozessors abhängt, da zwischen Sender und Empfänger ein Taktsignal bestehen muss. Es ist so zu sagen ein Protokoll, dass eine Primär- / Replikatkonfiguration erfordert, da ein Gerät die Kontrolle über ein anderes Gerät hat, um die Synchronität sicherzustellen.

### Asynchronous Portocol
Das Gegenteil von synchronem, asynchronem Protokoll arbeitet außerhalb der Beschränkungen eines Taktsignals und tritt zu jeder Zeit und in unregelmäßigen Intervallen auf. Wie oben erwähnt, ist das synchrone Protokoll mikroprozessorabhängig, was bedeutet, dass diese Protokolle häufig für die in einem Computer ablaufenden Prozesse verwendet werden. Bei asynchronen Prozessen sind sie aufgrund der fehlenden Abhängigkeit für Cloud-Umgebungen und Betriebssysteme weit verbreitet. Nach dem Senden von Informationen muss nicht auf eine Antwort des Empfängers gewartet werden.

### Single Receiver
Im Namen implizit bedeutet eine einzelne Empfängereinrichtung, dass jede Anforderung von einem Empfänger verarbeitet werden muss. Bei der Datenübertragung müssen Anforderungen daher gestaffelt sein, um empfangen zu werden, da sie nicht gleichzeitig empfangen werden können.

### Multiple Receivers
Mehrere Empfänger können mehrere Anforderungen verarbeiten, da jede Anforderung von null bis zu mehreren Empfängern verarbeitet werden kann. Beispielsweise werden in der Regel mehrere Empfänger in einem Saga-Muster verwendet, da es asynchron sein und gleichzeitig die Datenkonsistenz fördern muss.


## Monitoring
Anwendungen, die unter Verwendung der Mikroservice-Architektur entwickelt wurden, müssen aus denselben Gründen überwacht werden wie alle anderen Arten von verteilten Systemen: Das heißt, alle Systeme fallen schließlich aus.

* Überwachung von Containern und deren Inhalt
* Warungen zur Servicleistung, nicht zur Containerleistung
* Überwachung von Diensten welche felxibel und standortübergreifend sind
* Überwachung der APIs
* Überwachung über APIs

## APIs
API steht für Application Programming Interface, wobei das Schlüsselwort interface ist . APIs sind sozusagen die Türen , durch die Entwickler mit einer Anwendung interagieren können.
APIs gibt es seit den Anfängen des Computerbetriebs , die es Computern ermöglichen, Wiederholungsfunktionen aufzurufen, um das Aufblähen von Anwendungen zu verringern. Bei der Erörterung von APIs in der heutigen digitalen Wirtschaft geht es jedoch in der Regel um Web-APIs , die die B2B-Kommunikation erleichtern.
Allgemein gesagt, ermöglichen APIs Entwicklern, intern und extern eines von zwei Dingen auszuführen: Zugriff auf die Daten einer Anwendung oder Verwendung der Funktionen einer Anwendung . Letztendlich sind auf diese Weise die Elektronik, Anwendungen und Webseiten der Welt miteinander verbunden, um miteinander zu kommunizieren und zusammenzuarbeiten.

### Unterschiede zwischen APIs und Microservices¨
Microservice sind eine Architektur für Web - Anwendungen, bei denen die Funktionalität bis aufgeteilt in kleinem Web - Service.<br>

wohingegen<br>

APIs sind die Frameworks, über die Entwickler mit einer Webanwendung interagieren können.<br>
Der Reiz der APIs ist zweifach für die meisten Unternehmen, da APIs oft das Medium der Kommunikation zwischen Microservices sind, sondern auch , weil sie auch verwendet werden , kann eine Anwendung die Daten und Funktionalität an Dritte zu entlarven, was den Weg für leistungsstarke Integrationen.


**Fazit und Aussicht**, Die Durcharbeitung von 701.1 Modern Software Development gab mir ein besseres Verständnis darüber wie ein Microservices verschiedene Aufgaben handhabt und verwendet.

***

# Kapitel: 701.2 Standard Components and Platforms for Software (Status: In Arbeit)

**Weight**: 2 + 6 Bonuspunkte für die Kombination des Microservices, aus Topic 701.1, mit einer NoSQL Datenbank.

**Beschreibung** Aufstellung von Features und Konzepten von:
* Object storage
* Relational and NoSQL databases
* Message brokers and message queues
* Big data services
* Application runtimes / PaaS
* Content delivery networks

**Tagesziele**, Aufstellung der oben genannten Features und Konzepten.

**Vorgehen**, Informationen über die oben gennanten Features und Konzepte sammeln und in einer Aufstellung beschreiben.

**Beispiele und Arbeitsergebnisse**

TextTextText

**Fazit und Aussicht**, Die Durcharbeitung von 701.2 Standard Components and Platforms for Software gab mir ein besseres Verständnis darüber was für Features und Konzepte von einzelnen "Diensten" für Cloud Platformen verwendet werden.

***

# Kapitel: 701.3 Source Code Management (Status: In Arbeit)

**Weight**: 5

**Beschreibung** Aufstellung von Features und Eigenschaften von Git:
* Git conecpts and repository structure
* Manage files within a Git repostitory
* Manage branches and tags
* Work with remote repositories and branhes as well as submodules
* Merge files and branches
* Awareness of SVN and CVS, including concepts of centralized an distributed SCM solutions

**Tagesziele**, Aufstellung der oben genannten Features und Eingenschaften von Git.

**Vorgehen**, Informationen über die oben gennanten Features und Eigenschaften von Git sammeln und in einer Aufstellung beschreiben.

**Beispiele und Arbeitsergebnisse**

TextTextText

**Fazit und Aussicht**, Die Durcharbeitung von 701.3 Source Code Management gab mir ein besseres Verständnis darüber was für Features und Eigenschaften Git hat, wie Git diese verwendet und wie man sie selber anwenden.

***

# Kapitel: 702.1 Container Usage (Status: In Arbeit)

**Weight**: 7 (nur wer das Modul M300 nicht besucht hat + 7 Bonuspunkte). Es wird ein Beschrieb Linux Technologien und Container erwartet.

**Beschreibung** Gegenüberstellung welche Linux Technologien für Container verwendet werden.

**Tagesziele**, Erstellung einer Tabelle Linux - Container. 

**Vorgehen**, Studieren Background Linux Namespaces vs. Container, UnionFS vs. Container Layer, Unix Prozesse (Jobs) vs. Docker run/start/stop

**Beispiele und Arbeitsergebnisse**

| Linux          | Container      | Beschreibung      |
| -------------- | -------------- | ----------------- |
| Namespaces     | laufender Container | beim Starten des Containers wird in eine andere Linux Namespace gewechselt |
| UnionFS        | Image Layer         | Container Verwenden UnionFileSysteme um .... |
| Unix Prozesse  | run/start/stop      | docker run/start/stop Befehle ähneln dem .... Subsystem |

**Fazit und Aussicht**, Die Durcharbeitung von 702.1 Container Management gab mir ein besseres Verständnis über die Funktionsweise von Containern.

***

# Kapitel: 702.2 Container Deployment and Orchestration (Status: In Arbeit)

**Weight**: 5

**Beschreibung** Aufstellung von Cluster Features und Eigenschaften für Docker/Kubernetes:
* Understand the architecture and application model Kubernetes
* Definition of Deployments
* Definition of Services
* Definition of ReplicaSets
* Definition of Pods
* docker-compose
* docker
* kubectl

**Tagesziele**, Aufstellung der oben genannten Features und Eingenschaften von Git.

**Vorgehen**, Informationen über die oben gennanten Features und Eigenschaften von Git sammeln und in einer Aufstellung beschreiben.

**Beispiele und Arbeitsergebnisse**

TextTextText

**Fazit und Aussicht**, Die Durcharbeitung von 701.3 Source Code Management gab mir ein besseres Verständnis darüber was für Features und Eigenschaften Git hat, wie Git diese verwendet und wie man sie selber anwenden.

***

</details>


<details>
<summary>LPI Linux Essentials - Exam 010</summary>
<br>



</details>

</details>