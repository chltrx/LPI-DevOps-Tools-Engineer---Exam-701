# ![](https://www.lpice.eu/fileadmin/_processed_/csm_LPIC-DevOpsToolsEngineer_43de3c4735.jpg) LPI DevOps Tools Engineer - Exam 701

Dokumention über den Lern- und Entwicklungsprozesses mit Ausgesuchten Unterkapiteln aus dem LPI DevOps Tools Engineer - Exam 701 des Wahlmoduls W901 an der TBZ

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

<details>
<summary>Kapitel: 701.1 Modern Software Development</summary>
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
</details>

<details>
<summary>Kapitel: 701.2 Standard Components and Platforms for Software</summary>
<br>

# Kapitel: 701.2 Standard Components and Platforms for Software (Status: Abgeschlossen)

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

## Object storage
Objektspeicher ist eine Computer-Datenspeicherarchitektur, die Daten als Objekte verwaltet. Im Gegensatz zu anderen Speicherarchitekturen wie Dateisystemen, die Daten als Dateihierarchie verwalten, und Blockspeicher, die Daten Blöcke innerhalb von Sektoren und Spuren verwaltet. Jedes Objekt enthält normalerweise die Daten selbst, eine variable Menge an Metadaten und einen global eindeutigen Bezeichner. Die Objektspeicherung kann auf mehreren Ebenen implementiert werden, einschließlich der Geräteebene (Objektspeichergerät), der Systemebene und der Schnittstellenebene. In jedem Fall versucht der Objektspeicher Funktionen zu ermöglichen die von anderen Speicherarchitekturen nicht adressiert werden wie z.B. Schnittstellen, die von der Anwendung direkt programmiert werden können, einen Namespace, der mehrere Instanzen physischer Hardware umfassen kann und Datenverwaltungsfunktionen wie Datenreplikation und Datenverteilung auf Objektebene Granularität.
Objektspeichersysteme ermöglichen die Speicherung großer Mengen an unstrukturierten Daten. Die Objektspeicherung wird beispielsweise zum Speichern von Fotos auf Facebook, von Titeln auf Spotify oder von Dateien in Online-Collaboration-Diensten wie Dropbox verwendet.


## Relational and NoSQL databases
### Data models
Mit einer NoSQL-Datenbank kann man eine Anwendung erstellen, ohne zuerst das Schema definieren zu müssen, im Gegensatz zu relationalen Datenbanken, mit denen man das Schema definieren muss, bevor man dem System Daten hinzufügen kann. Kein vordefiniertes Schema erleichtert die Aktualisierung von NoSQL-Datenbanken erheblich, da sich Ihre Daten und Anforderungen ändern.

### Data structure
Relationale Datenbanken wurden in einer Ära erstellt, in der Daten durch ihre Beziehungen fair strukturiert und klar definiert waren. NoSQL-Datenbanken sind so konzipiert, dass sie unstrukturierte Daten (z. B. Texte, Social Media-Posts, Videos, E-Mails) verarbeiten,die einen Großteil der heute vorhandenen Daten ausmachen.

### Scaling
Die Skalierung einer NoSQL-Datenbank ist viel billiger als die einer relationalen Datenbank, da man durch Skalierung über billige Commodity-Server Kapazität hinzufügen kann. Relationale Datenbanken erfordern andererseits einen einzelnen Server, um ihre gesamte Datenbank zu hosten. Um zu skalieren, muss man einen größeren, teureren Server kaufen.

### Development model
NoSQL-Datenbanken sind Open-Source-Datenbanken, während relationale Datenbanken in der Regel Closed-Source-Datenbanken sind und Lizenzgebühren für die Verwendung ihrer Software anfallen. Mit NoSQL kann man in ein Projekt einsteigen, ohne vorher viel in Software-Gebühren investieren zu müssen.


## Message brokers and message queues
### Message broker
Ein Message broker ist ein Vermittler Computerprogrammmodul, dass eine Nachricht von dem formalen Nachrichtenprotokoll von dem Sender zu dem formalen Nachrichtenprotokoll des Empfängers übersetzt. Nachrichtenbroker sind Elemente in Telekommunikations- oder Computernetzwerken, in denen Softwareanwendungen durch den Austausch von formal definierten Nachrichten kommunizieren. Nachrichtenbroker sind ein Baustein für nachrichtenorientierte Middleware (MOM), ersetzen jedoch in der Regel keine herkömmliche Middleware wie MOM und Remote Procedure Call (RPC).

### Message queues
Anwendungen, die Message-Queuing-Verfahren einsetzen, weisen folgende Hauptmerkmale auf:

* Es bestehen keine Direktverbindungen zwischen Programmen.
* Die Kommunikation zwischen Programmen kann zeitunabhängig erfolgen.
* Die Arbeit kann von kleinen eigenständigen Programmen ausgeführt werden.
* Die Kommunikation kann ereignisgesteuert sein.
* Anwendungen können Nachrichten Prioritäten zuweisen.
* Sicherheit.
* Datenintegrität.
* Unterstützung bei der Wiederherstellung.

Message-Queuing stellt ein Verfahren für die indirekte Kommunikation zwischen Programmen dar. Dieses Verfahren kann in jeder Anwendung eingesetzt werden, in der Programme miteinander kommunizieren. Die Kommunikation erfolgt, indem ein Programm Nachrichten in eine (einem Warteschlangenmanager zugehörige) Warteschlange einreiht und ein anderes Programm die Nachrichten aus der Warteschlange abruft.
Programme können Nachrichten abrufen, die von anderen Programmen in eine Warteschlange eingereiht wurden. Die anderen Programme können mit demselben Warteschlangenmanager wie das empfangende Programm oder aber mit einem anderen Warteschlangenmanager verbunden sein. Dieser andere Warteschlangenmanager kann sich auf einem anderen System, in einem anderen Computersystem oder sogar in einem anderen Unternehmen befinden.


## Big data services
Um Big Data zu verstehen, ist es hilfreich, den geschichtlichen Hintergrund zu kennen. Das ist die Definition von Gartner, die etwa aus dem Jahr 2001 stammt und nach wie vor die gängigste ist: Unter Big Data versteht man Daten, die in großer Vielfalt, in großen Mengen und mit hoher Geschwindigkeit anfallen. Dies ist auch als die drei V-Begriffe bekannt (Variety, Volume, Velocity).
Einfach gesagt: Mit Big Data bezeichnet man größere und komplexere Datensätze, vor allem von neuen Datenquellen. Diese Datensätze sind so umfangreich, dass klassische Datenverarbeitungssoftware sie nicht verwalten kann. Aber mit diesen massiven Datenvolumina können Sie geschäftliche Probleme angehen, die Sie bislang nicht lösen konnten.

### Volume
Die Menge an Daten ist wichtig. Bei Big Data müssen Sie große Mengen an unstrukturierten Daten mit geringer Dichte verarbeiten. Dabei kann es sich um Daten mit unbekanntem Wert handeln, z. B. Daten-Feeds von Twitter, Clickstreams von einer Webseite oder mobilen App oder Daten von Gerätesensoren. Für einige Unternehmen können das etliche Terabytes an Daten sein. Für andere Hunderte von Petabytes.

### Velocity
Die Geschwindigkeit ist die Schnelligkeitsrate, mit der Daten empfangen werden und mit der (vielleicht) auf sie reagiert wird. Im Normalfall fließt die höchste Geschwindigkeit von Daten direkt in den Speicher und wird nicht auf eine Festplatte geschrieben. Einige internetfähige, intelligente Produkte arbeiten in Echtzeit oder beinahe in Echtzeit. Für sie sind Auswertungen und Aktionen in Echtzeit erforderlich.

### Variety
Vielfalt bezieht sich auf die zahlreichen verfügbaren Datentypen. Traditionelle Datentypen waren strukturiert und ideal für relationale Datenbanken geeignet. Durch die Zunahme von Big Data gibt es nun neue, unstrukturierte Datentypen. Unstrukturierte und semistrukturierte Datentypen wie Text, Audio und Video erfordern zusätzliche Vorabverarbeitung, um die Bedeutung und die unterstützenden Metadaten zu gewinnen.


## Application runtimes / PaaS
### Application runtimes
Eine Laufzeitumgebung beschreibt die zur Laufzeit von Computerprogrammen verfügbaren und festgelegten Voraussetzungen eines bestimmten Laufzeitsystems. Dieses ist durch die elementaren Bestandteile der Programmiersprache wie das Verhalten von Sprachkonstrukten und weitere Funktionen wie Typprüfung, Debugging, Codegenerierung und -optimierung definiert. Zur Laufzeitumgebung gehören weiterhin Laufzeitbibliothek, Standardbibliotheken, Programmierschnittstellen, Laufzeit-Variablen sowie auf Hard- und Softwarekomponenten über Betriebssystemfunktionen.

### PaaS
Als Platform as a Service (PaaS) bezeichnet man eine Dienstleistung, die in der Cloud eine Computer-Plattform für Entwickler von Webanwendungen zur Verfügung stellt. Dabei kann es sich sowohl um schnell einsetzbare Laufzeitumgebungen (typischerweise für Webanwendungen), aber auch um Entwicklungsumgebungen handeln, die mit geringem administrativem Aufwand und ohne Anschaffung der darunterliegenden Hardware und Software genutzt werden können. Sie unterstützen den gesamten Software-Lebenszyklus vom Design über die Entwicklung, den Test, die Auslieferung bis hin zum Betrieb der Webanwendungen über das Internet. Platform as a Service ist ein Teil von Everything as a Service.
Einige Angebote umfassen auch Dienste zur kollaborativen Arbeit und Versionierung, zum Monitoring und für die Sicherheit oder Middleware-Dienste zum Speichern von Daten oder für die Kommunikation zwischen Anwendungen. PaaS-Angebote bauen auf einer skalierbaren Infrastruktur (IaaS) von Speicher und Rechenleistung auf und können somit ebenfalls skalieren. Aufbauend auf einer PaaS-Umgebung können Software as a Service (SaaS)-Angebote entstehen. Somit ist PaaS die mittlere Schicht im Cloud Stack.


## Content delivery networks
Ein Content Delivery Network (CDN), oder auch Content Distribution Network genannt, ist ein Netz regional verteilter und über das Internet verbundener Server, mit dem Inhalte – insbesondere große Mediendateien – ausgeliefert werden. Ein CDN stellt skalierende Speicher- und Auslieferungskapazitäten zur Verfügung und gewährleistet auch bei großen Lastspitzen einen optimalen Datendurchsatz.
CDN-Knoten sind auf viele Orte verteilt und oft auch auf viele Backbones. Sie arbeiten zusammen, um Anfragen (Requests) von End-Nutzern nach Inhalten (Content) möglichst ökonomisch zu bedienen. Einzelne Standorte werden als PoP (Point of Presence) bezeichnet und bestehen aus Server-Clustern.
Im Hintergrund (Transparent) werden die Daten im Netz so vorgehalten (Caching), dass die jeweilige Auslieferung entweder möglichst schnell geht (Performance-Optimierung) oder möglichst wenig Bandbreite verbraucht (Kosten-Optimierung), oder beides zugleich.
Große CDNs unterhalten tausende Knoten mit zehntausenden Servern.

### Funktionsweise
Das CDN besteht zunächst aus einem Ursprungsserver, auf dem der Inhalteanbieter die zu verteilenden Inhalte ablegt, einer großen Zahl an Replica-Servern, die Kopien dieser Inhalte vorhalten, und einem Distributionssystem, das die Inhalte auf den Replica-Servern verteilt. Für die Umleitung der Benutzeranfragen auf die einzelnen Replica-Server ist ein Request-Routing-System zuständig, welches sich dabei auf verschiedene Kennzahlen über diese Server stützt, die ihm vom Accounting-System geliefert werden.
Sendet ein Client eine Anfrage an das CDN, dann wählt das Request-Routing-System einen geeigneten Replica-Server. Bei der Auswahl bezieht es Kennzahlen über deren aktuelle Belastung (zum Beispiel CPU-Auslastung, Anzahl der aktiven Verbindungen) und über die Netzwerkverbindung zwischen Client und Server (zum Beispiel geographische Entfernung, Latenzzeit, Übertragungsrate), seltener über die Identität des Clients (zum Beispiel Unterscheidung zwischen Standard- und Premium-User) mit ein, die ihm durch das Accounting-System zur Verfügung gestellt werden.
Nach Auswahl des Servers muss die Benutzeranfrage nun umgeleitet werden. Das am häufigsten eingesetzte Verfahren dafür ist DNS-basiertes Request Routing. Dabei werden Anfragen des Clients an einen vom CDN bereitgestellten DNS-Server weitergeleitet, welcher die IP-Adresse des Replica-Servers zurückgibt. Alternativ dazu kann auch ein HTTP-Statuscode 302 die Weiterleitung auf einen anderen Webserver veranlassen.

**Fazit und Aussicht**, Die Durcharbeitung von 701.2 Standard Components and Platforms for Software gab mir ein besseres Verständnis darüber was für Features und Konzepte von einzelnen "Diensten" für Cloud Platformen verwendet werden.

***
</details>

<details>
<summary>Kapitel: 701.3 Source Code Management</summary>
<br>

# Kapitel: 701.3 Source Code Management (Status: Abgeschlossen)

**Weight**: 5

**Beschreibung** Aufstellung von Features und Eigenschaften von Git:
* Git concepts and repository structure
* Manage files within a Git repostitory
* Manage branches and tags
* Work with remote repositories and branches as well as submodules
* Merge files and branches
* Awareness of SVN and CVS, including concepts of centralized and distributed SCM solutions

**Tagesziele**, Aufstellung der oben genannten Features und Eingenschaften von Git.

**Vorgehen**, Informationen über die oben gennanten Features und Eigenschaften von Git sammeln und in einer Aufstellung beschreiben.

**Beispiele und Arbeitsergebnisse**

## How Git works ++
Ein Git- Repository ist einfach eine Datenbank, die alle Informationen enthält, die zum Speichern und Verwalten der Revisionen und des Verlaufs eines Projekts erforderlich sind. In Git behält ein Repository wie bei den meisten Versionskontrollsystemen eine vollständige Kopie des gesamten Projekts während seiner gesamten Lebensdauer. Im Gegensatz zu den meisten anderen VCSs bietet das Git-Repository jedoch nicht nur eine vollständige Arbeitskopie aller Dateien im Repository, sondern auch eine Kopie des Repositorys selbst, mit dem gearbeitet werden soll.<br>

Git verwaltet eine Reihe von Konfigurationswerten in jedem Repository wie z. B. den Namen und die E-Mail-Adresse des Repository-Benutzers. Im Gegensatz zu Dateidaten und anderen Repository-Metadaten werden Konfigurationseinstellungen während eines Klon- oder Dupliziervorgangs nicht von einem Repository auf ein anderes übertragen. Stattdessen verwaltet und überprüft Git die Konfigurations- und Einrichtungsinformationen auf Site-, Benutzer- und Repository-Basis.<br>

In einem Repository verwaltet Git zwei primäre Datenstrukturen, den Objektspeicher und den Index . Alle diese Repository-Daten werden im Stammverzeichnis Ihres Arbeitsverzeichnisses in einem versteckten Unterverzeichnis mit dem Namen .git gespeichert. <br>

Der Objektspeicher soll während einer Klonoperation als Teil des Mechanismus, der ein vollständig verteiltes VCS unterstützt, effizient kopiert werden. Beim Index handelt es sich um vorübergehende Informationen, die für ein Repository privat sind und bei Bedarf erstellt oder geändert werden können.

### Blobs
Jede Version einer Datei wird als Blob dargestellt . " Blob " ist eine Abkürzung für " binäres großes Objekt ", ein Begriff, der üblicherweise beim Rechnen verwendet wird, um sich auf eine Variable oder Datei zu beziehen, die beliebige Daten enthalten kann und deren interne Struktur vom Programm ignoriert wird. Ein Klecks wird als undurchsichtig behandelt. Ein Blob enthält die Daten einer Datei, jedoch keine Metadaten über die Datei oder sogar ihren Namen.

### Trees
Ein Baum - Objekt stellt eine Ebene der Verzeichnisinformationen. Es zeichnet Blob-IDs, Pfadnamen und einige Metadaten auffür alle Dateien in einem Verzeichnis. Es kann auch rekursiv auf andere (Unter-) Baumobjekte verweisen und so eine vollständige Hierarchie von Dateien und Unterverzeichnissen erstellen.

### Commits
Ein Commit enthält Metadaten für jede im Repository eingeführte Änderung, einschließlich Autor, Datum und Protokollnachricht. Jedes Commit verweist auf ein Baumobjekt, das in einem vollständigen Schnappschuss Folgendes erfasst: 

    Der Status des Repositorys zum Zeitpunkt der Ausführung des Commits. 
    Das ursprüngliche Commit oder Root-Commit hat kein übergeordnetes Element. 

### Tags
Ein Tag- Objekt weist einem bestimmten Objekt, normalerweise einem Commit, einen beliebigen, jedoch vermutlich von Menschen lesbaren Namen zu. Obwohl es 9da581d910c9c4ac93557ca4859e767f5caf5169 sich um ein genaues und genau definiertes Commit handelt, ist ein vertrauterer Tag-Name Ver-1.0-Alphamöglicherweise sinnvoller.

### Index
Der Index ist eine temporäre und dynamische Binärdatei, die die Verzeichnisstruktur des gesamten Repository beschreibt. Insbesondere erfasst der Index zu einem bestimmten Zeitpunkt eine Version der Gesamtstruktur des Projekts. Der Status des Projekts kann durch ein Festschreiben und einen Baum von einem beliebigen Punkt in der Projektgeschichte aus dargestellt werden, oder es kann sich um einen zukünftigen Status handeln, auf den Sie sich aktiv hinentwickeln.<br>

Eines der wichtigsten Unterscheidungsmerkmale von Git ist, dass man den Inhalt des Index in methodischen, genau definierten Schritten ändern kann. Der Index ermöglicht eine Trennung zwischen inkrementellen Entwicklungsschritten und der Übernahme dieser Änderungen.

### Content-Addressable Names
Der Git-Objektspeicherist als inhaltsadressierbares Speichersystem organisiert und implementiert. Insbesondere hat jedes Objekt im Objektspeicher einen eindeutigen Namen, der durch Anwenden von SHA1 auf den Inhalt des Objekts erzeugt wird und einen SHA1-Hashwert ergibt. Da der vollständige Inhalt eines Objekts zum Hash-Wert beiträgt und davon ausgegangen wird, dass der Hash-Wert für diesen bestimmten Inhalt eindeutig ist, ist der SHA1-Hash ein ausreichender Index oder Name für dieses Objekt in der Objektdatenbank. Bei jeder geringfügigen Änderung an einer Datei ändert sich der SHA1-Hash, wodurch die neue Version der Datei separat indiziert wird.

### Content-Tracking-System
Es ist wichtig, Git nicht nur als Versionskontrollsystem zu betrachten denn Git ist auch ein Content-Tracking-System. Diese noch so subtile Unterscheidung leitet einen Großteil des Designs von Git und ist möglicherweise der Hauptgrund, warum Git interne Datenmanipulationen relativ einfach durchführen kann. Dies ist jedoch möglicherweise auch eines der am schwierigsten zu erfassenden Konzepte für neue Git-Benutzer. Daher lohnt sich eine Einführung.<br>

Der Objektspeicher von Git basiert auf der Hash-Berechnung des Inhalts seiner Objekte, nicht auf den Datei- oder Verzeichnisnamen aus dem ursprünglichen Dateilayout des Benutzers. Wenn Git eine Datei in den Objektspeicher legt, basiert dies auf dem Hash der Daten und nicht auf dem Namen der Datei. Tatsächlich verfolgt Git keine Datei- oder Verzeichnisnamen, die Dateien auf sekundäre Weise zugeordnet sind. Auch hier verfolgt Git Inhalte anstelle von Dateien.<br>

Wenn zwei separate Dateien in zwei verschiedenen Verzeichnissen genau denselben Inhalt haben, speichert Git eine einzige Kopie dieses Inhalts als Blob im Objektspeicher. Git berechnet den Hash-Code jeder Datei ausschließlich anhand ihres Inhalts, stellt fest, dass die Dateien dieselben SHA1-Werte und damit denselben Inhalt haben, und platziert das Blob-Objekt in dem durch diesen SHA1-Wert indizierten Objektspeicher. Beide Dateien im Projekt verwenden, unabhängig davon, wo sie sich in der Verzeichnisstruktur des Benutzers befinden, dasselbe Objekt für den Inhalt.<br>

Wenn sich eine dieser Dateien ändert, berechnet Git einen neuen SHA1 und stellt fest, dass es sich nun um ein anderes Blob-Objekt handelt und fügt den neuen Blob dem Objektspeicher hinzu. Der ursprüngliche Blob verbleibt im Objektspeicher, damit die unveränderte Datei verwendet werden kann.<br>

Dazu speichert die interne Datenbank von Git effizient jede Version jeder Datei - nicht deren Unterschiede - während die Dateien von einer Revision zur nächsten wechseln. Da Git den Hash des gesamten Inhalts einer Datei als Namen für diese Datei verwendet, muss jede vollständige Kopie der Datei bearbeitet werden. Es kann seine Arbeit oder seine Objektspeichereinträge nicht nur auf einen Teil des Dateiinhalts oder auf die Unterschiede zwischen zwei Revisionen dieser Datei stützen.<br>

Die typische Benutzeransicht einer Datei, die Revisionen enthält und von einer Revision zur nächsten zu wechseln scheint, ist einfach ein Artefakt. Git berechnet diesen Verlauf als eine Reihe von Änderungen zwischen verschiedenen Blobs mit unterschiedlichen Hashes, anstatt einen Dateinamen und eine Reihe von Unterschieden direkt zu speichern. Es mag seltsam erscheinen, aber diese Funktion ermöglicht es Git, bestimmte Aufgaben mit Leichtigkeit auszuführen.

### SVN vs. CVS

| Nr.            | Vergelichselement | CVS            | SVN               |
| -------------- | -------------- | ----------------- | ----------------- |
| 1              | Repository-Format |CVS basiert auf RCS-Dateien der Versionskontrolle. Jede mit CVS verbundene Datei ist eine gewöhnliche Datei, die einige zusätzliche Informationen enthält. Es ist ganz natürlich, dass der Baum dieser Dateien den Dateibaum im lokalen Verzeichnis wiederholt. Daher sollten Sie mit CVS keine Angst vor Datenverlust haben und können RCS-Dateien bei Bedarf problemlos korrigieren.|Die Basis von SVN ist eine relationale Datenbank (BerkleyDB), entweder ein Satz von Binärdateien (FS_FS). Dies behebt einerseits viele Probleme (z.B. gleichzeitiger Zugriff über die Dateifreigabe) und ermöglicht neue Funktionen (z.B. Transaktionen bei der Betriebsleistung). Andererseits ist die Datenspeicherung jetzt nicht transparent oder steht zumindest nicht für Benutzereingriffe zur Verfügung. Aus diesem Grund werden die Hilfsprogramme zum "Bereinigen" und "Wiederherstellen" des Repositorys (der Datenbank) bereitgestellt.|
| 2              | Geschwindigkeit |CVS arbeitet langsamer.|Insgesamt arbeitet SVN aufgrund einiger konstruktiver Lösungen wirklich schneller als CVS. Es überträgt weniger Informationen über das Netzwerk und unterstützt mehr Vorgänge im Offline-Modus. Es gibt jedoch die Umkehrung der Medaille. Die Geschwindigkeitssteigerung erfolgt im Wesentlichen auf Kosten der vollständigen Sicherung aller Arbeitsdateien auf Ihrem Computer.|
| 3              | Tags & Branches |Diese werden normal und ordnungsgemäß umgesetzt.	|Die SVN-Entwickler behaupten mit Stolz, dass sie durch die Arbeit mit Tags und Zweigen drei Messungen eliminiert haben. In der Praxis bedeutet dies, dass beide Konzepte durch die Möglichkeit ersetzt wurden, Dateien oder Verzeichnisse im Repository zu kopieren, wobei der Änderungsverlauf gespeichert wurde. Das heißt, sowohl die Tag-Erstellung als auch die Zweig-Erstellung werden durch das Kopieren innerhalb des Repository ersetzt. Aus Sicht der SVN-Entwickler ist dies eine sehr elegante Entscheidung, die das Leben vereinfacht. Wir sind jedoch der Meinung, dass es nichts gibt, worauf wir stolz sein können. Bei Zweigen ist alles nicht so schlimm, jetzt sind Zweige nichts anderes als separate Ordner im Repository, die früher miteinander verbunden waren. Was Tags betrifft, ist alles viel schlimmer. Jetzt können Sie keinen Code mehr markieren, diese Funktion fehlt einfach. Bestimmt, Zum Teil wird dies durch die universelle Nummerierung der Dateien in SVN ausgeglichen, dh das gesamte Repository erhält die Versionsnummer, jedoch nicht jede einzelne Datei. Wir nehmen jedoch an, dass Sie nicht leugnen werden, dass es nicht sehr bequem ist, eine vierstellige Zahl anstelle eines symbolischen Tags zu speichern.|  
| 4              | Metadaten       |CVS erlaubt nur das Speichern von Dateien und sonst nichts.	|SVN erlaubt es, eine beliebige Anzahl aller möglichen benannten Attribute an eine Datei anzuhängen.|  
| 5              | Datentypen      |CVS war ursprünglich zur Speicherung von Textdaten gedacht. Aus diesem Grund ist die Speicherung anderer Dateien (binär, Unicode) nicht trivial und erfordert spezielle Informationen sowie Anpassungen auf Server- oder Clientseite.|SVN bearbeitet alle Dateitypen und benötigt keine Anweisungen.|  
| 6              | Rollback        |CVS ermöglicht das Rollback von Commits im Repository, auch wenn dies einige Zeit in Anspruch nimmt (jede Datei sollte unabhängig verarbeitet werden).|SVN erlaubt kein Rollback des Commits. Die Autoren schlagen vor, einen guten Repository-Status an das Ende des Trunks zu kopieren, um einen fehlerhaften Commit zu überschreiben. Ein schlechtes Commit selbst verbleibt jedoch im Repository.|  
| 7              | Transaktionen   |In CVS fehlt die Unterstützung von Transaktionen nach dem Prinzip "Alles oder Nichts" vollständig. Wenn Sie beispielsweise mehrere Dateien einchecken (auf den Server übertragen), wird der Vorgang möglicherweise nur für einige dieser Dateien abgeschlossen und für den Rest nicht (z. B. aufgrund von Konflikten). In der Regel ist es ausreichend, die Situation zu korrigieren und den Vorgang für die verbleibenden Dateien (nicht für alle Dateien) zu wiederholen. Das heißt, die Dateien werden in zwei Schritten eingecheckt. Es wurden keine Fälle von Lagerschäden aufgrund des Fehlens dieser Funktionalität beobachtet.|SVN unterstützt Transaktionen nach dem Prinzip "Alles oder Nichts".| 
| 8              | Verfügbarkeit   | CVS wird überall dort unterstützt, wo man es benötigt.	|SVN ist noch nicht so weit verbreitet, so dass es Stellen gibt, an denen es noch nicht implementiert ist.|  
| 9              | Interne Architektur und Code |CVS ist ein sehr altes System. Ursprünglich wurde CVS als eine Reihe von Skripten rund um die ausführbare RCS-Datei geschrieben. Später wurde dies in eine einzelne ausführbare Datei gepackt. Die interne Struktur des Codes ist jedoch schlecht und enthält viele historische Korrekturen. Bis jetzt gab es mehrere Versuche, CVS von Grund auf neu zu schreiben, aber wie bekannt, ohne Erfolg. Wir haben persönlich versucht, Client-Code zu extrahieren, um eine bessere Integration zwischen Plug-In und CVS zu erreichen, aber ohne Erfolg. Momentan glauben wir nicht, dass CVS in seiner Funktionalität zu stark zunimmt.	|Subversion-Autoren haben wirklich einige Zeit mit der internen SVN-Architektur verbracht. Ich weiß immer noch nicht, wie gut einige Entscheidungen sind, die sie treffen. Aber eines ist klar, der Code ist gut erweiterbar und zukünftige Verbesserungen stehen an.|   


**Fazit und Aussicht**, Die Durcharbeitung von 701.3 Source Code Management gab mir ein besseres Verständnis darüber was für Features und Eigenschaften Git hat, wie Git diese verwendet und wie man sie selber anwenden kann.

***
</details>

<details>
<summary>Kapitel: 702.1 Container Usage</summary>
<br>

# Kapitel: 702.1 Container Usage (Status: Abgeschlossen)

**Weight**: 7 (nur wer das Modul M300 nicht besucht hat + 7 Bonuspunkte). Es wird ein Beschrieb Linux Technologien und Container erwartet.

**Beschreibung** Auflistung welche Linux Technologien für Container verwendet werden.
* Namespaces
* Container
* UnionFS
* Container Layers
* Linux jobs & processes
* Docker run/start/stop

**Tagesziele**, Erstellung einer Auflistung zu Linux Container Technologien. 

**Vorgehen**, Hintergrund zu Linux Namespaces, Container, UnionFS, Container Layer, Unix Prozesse (Jobs) und Docker run/start/stop sammeln und auflisten.

**Beispiele und Arbeitsergebnisse**

## Namespaces
Namespaces sind eine Funktion des Linux-Kernels , mit der Kernelressourcen so partitioniert werden, dass für einen Satz von Prozessen ein Satz von Ressourcen und für einen anderen Satz von Prozessen ein anderer Satz von Ressourcen angezeigt wird. Die Funktion verwendet denselben Namespace für diese Ressourcen in den verschiedenen Prozessgruppen, diese Namen beziehen sich jedoch auf unterschiedliche Ressourcen. Beispiele für Ressourcennamen, die in mehreren Bereichen vorhanden sein können, damit die benannten Ressourcen partitioniert werden, sind Prozess-IDs, Hostnamen, Benutzer-IDs, Dateinamen und einige Namen, die dem Netzwerkzugriff und der Interprozesskommunikation zugeordnet sind .
Namespaces sind ein grundlegender Aspekt von Containern unter Linux.
Der Begriff "Namespace" wird häufig für eine Art Namespace (z. B. Prozess-ID) sowie für einen bestimmten Namensraum verwendet.
Ein Linux-System beginnt mit einem einzelnen Namespace jedes Typs, der von allen Prozessen verwendet wird. Prozesse können zusätzliche Namespaces erstellen und verschiedene Namespaces verbinden.


## Container
Ein Linux® Container besteht aus einem oder mehreren Prozessen, die vom Rest des Systems isoliert sind. Alle zur Ausführung notwendigen Dateien werden über ein separates Image bereitgestellt, d. h. Linux-Container sind von der Entwicklung über die Testphase bis hin zur Produktion stets portierbar und konsistent. Sie sind deshalb viel schneller als Entwicklungs-Pipelines, bei denen herkömmliche Testumgebungen repliziert werden. Wegen ihrer Beliebtheit und Benutzerfreundlichkeit sind Container ebenfalls ein wichtiger Bestandteil der IT-Sicherheit.


## UnionFS
UnionFS ist ein Dateisystem, das ursprünglich für das Plan-9-Betriebssystem entwickelt wurde. Es wurde dazu verwendet, Prozessen eigene Namensräume innerhalb der Dateisysteme zuzuweisen.
Mittels UnionFS werden die Dateien verschiedener Dateisysteme zu einem einzigen logischen Dateisystem vereinigt, d. h. Dateien, die in den getrennten Dateisystemen im gleichen Verzeichnis liegen, werden durch UnionFS im selben Verzeichnis angezeigt. Dabei werden den einzelnen beteiligten Hierarchien Prioritäten zugeordnet, so dass eine eindeutige Zuordnung auch im Falle gleicher Dateinamen gewährleistet ist.
Ein heutiger Einsatzzweck ist die Überlagerung von schreibgeschützten Dateisystemen mit RAM-Disks. Damit wird erreicht, dass Benutzer von Live-CDs lokale Dateien ändern können. Es kommt z. B. bei der Linux-Variante des Asus Eee PC zum Einsatz.
UnionFS wurde sowohl für Linux als auch für diverse BSD-Varianten implementiert und wird zum Beispiel bei Live-CDs von Damn Small Linux erfolgreich eingesetzt.


## Container Layers
Docker-Container sind Bausteine ​​für Anwendungen. Jeder Container ist ein Bild mit einer read/write Ebene über einer Reihe schreibgeschützter Ebenen.
Diese Ebenen (auch als Zwischenbilder bezeichnet) werden generiert, wenn die Befehle in der Docker-Datei während der Erstellung des Docker-Images ausgeführt werden.
Hier ist beispielsweise eine Docker-Datei zum Erstellen eines Web-App- Images für node.js. Es werden die Befehle angezeigt, die zum Erstellen des Abbilds ausgeführt werden.

    FROM node:argon
    # Create app directory
    RUN mkdir -p /usr/src/app
    WORKDIR /usr/src/app
    # Install app dependencies
    COPY package.json /usr/src/app/
    RUN npm install
    # Bundle app source
    COPY . /usr/src/app
    EXPOSE 8080
    CMD [ "npm", "start" ]

Wenn Docker den Container aus der obigen Docker-Datei erstellt, entspricht jeder Schritt einem Befehl, der in der Docker-Datei ausgeführt wird. Jede Ebene besteht aus der Datei, die durch Ausführen dieses Befehls generiert wurde. Zusammen mit jedem Schritt wird der erstellte Layer durch seine zufällig generierte ID dargestellt. Beispielsweise lautet die Layer-ID für Schritt 1 530c750a346e.

    $ docker build -t expressweb .
    Step 1 : FROM node:argon
    argon: Pulling from library/node...
    ...
    Status: Downloaded newer image for node:argon
     ---> 530c750a346e
    Step 2 : RUN mkdir -p /usr/src/app
     ---> Running in 5090fde23e44
     ---> 7184cc184ef8
    Removing intermediate container 5090fde23e44
    Step 3 : WORKDIR /usr/src/app
     ---> Running in 2987746b5fba
     ---> 86c81d89b023
    Removing intermediate container 2987746b5fba
    Step 4 : COPY package.json /usr/src/app/
     ---> 334d93a151ee
    Removing intermediate container a678c817e467
    Step 5 : RUN npm install
     ---> Running in 31ee9721cccb
     ---> ecf7275feff3
    Removing intermediate container 31ee9721cccb
    Step 6 : COPY . /usr/src/app
     ---> 995a21532fce
    Removing intermediate container a3b7591bf46d
    Step 7 : EXPOSE 8080
     ---> Running in fddb8afb98d7
     ---> e9539311a23e
    Removing intermediate container fddb8afb98d7
    Step 8 : CMD npm start
     ---> Running in a262fd016da6
     ---> fdd93d9c2c60
    Removing intermediate container a262fd016da6
    Successfully built fdd93d9c2c60

Sobald das build erstellt wurde, können Sie mit dem Befehl docker history alle Ebenen anzeigen, aus denen das build besteht. Die Spalte "build" (d.h. Zwischenbild oder Ebene) zeigt die zufällig generierte UUID, die mit dieser Ebene korreliert.

    docker history <image>
    $ docker history expressweb
    IMAGE         CREATED    CREATED BY                       SIZE      
    fdd93d9c2c60  2 days ago /bin/sh -c CMD ["npm" "start"]   0 B
    e9539311a23e  2 days ago /bin/sh -c EXPOSE 8080/tcp       0 B
    995a21532fce  2 days ago /bin/sh -c COPY dir:50ab47bff7   760 B
    ecf7275feff3  2 days ago /bin/sh -c npm install           3.439 MB
    334d93a151ee  2 days ago /bin/sh -c COPY file:551095e67   265 B
    86c81d89b023  2 days ago /bin/sh -c WORKDIR /usr/src/app  0 B
    7184cc184ef8  2 days ago /bin/sh -c mkdir -p /usr/src/app 0 B
    530c750a346e  2 days ago /bin/sh -c CMD ["node"]          0 B

Jede Zeile unter "Image" oben ist ein Layer eines Containers.


## Linux jobs & processes
Ein Prozess ist ein laufendes Programm mit einem eigenen Adressraum.
Ein Job ist ein Konzept, das von der Shell verwendet wird. Jedes Programm, dass man interaktiv startet und das sich nicht beendet (d.h. kein Daemon), ist ein Job. Wenn man ein interaktives Programm ausführt, kann man Ctrl+Z drücken, um es anzuhalten. Dann kann es wieder im Vordergrund (mit fg) oder im Hintergrund (mit bg) starten.

Während das Programm angehalten wird oder im Hintergrund ausgeführt wird, kann man ein anderes Programm starten. In diesem Fall werden zwei Jobs ausgeführt. Man kann auch ein Programm im Hintergrund laufen lassen, indem man ein „&“ wie folgt anhängt

    program &

. Dieses Programm wird als Hintergrundjob ausgeführt werden.

## Docker run/start/stop
### docker run
Der Grundbefehl docker runhat folgende Form:

    docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]

Der docker runBefehl muss ein IMAGE angeben , von dem der Container abgeleitet werden soll. Ein Bildentwickler kann Bildstandards definieren, die sich auf Folgendes beziehen:

* Ob er im Hintergrund oder im Vordergrund läuft
* Container identifikation
* Netzwerkeinstellungen
* Laufzeitbeschränkungen für CPU und Speicher

Mit dem kann docker run [OPTIONS] ein operator die von einem Entwickler eingestellten buildvorgaben ergänzen oder überschreiben. Außerdem können operatoren fast alle Standardeinstellungen überschreiben, die von der Docker-Laufzeit selbst festgelegt wurden. Aufgrund der Möglichkeit des operators, die Standardeinstellungen für Image- und Docker-Laufzeit zu überschreiben, verfügt run über mehr Optionen als jeder andere docker Befehl.

### docker start
Dieser Befehl strartet einen oder mehrere gestoppte Container.<br>
Der Grundbefehl docker runhat folgende Form:

    docker start [OPTIONS] CONTAINER [CONTAINER...]

| Name, Kurzform          | Beschreibung      |
| -------------- | -------------- |
| --attach , -a     | STDOUT / STDERR anhängen und Signale weiterleiten |
| --checkpoint	        | Wiederherstellen von diesem Checkpoint |
| --checkpoint-dir  | Verwenden von einem benutzerdefinierten Checkpoint-Speicherverzeichnis     |
| --detach-keys	  | Überschreibt die Tastenfolge zum Entfernen eines Containers     |
| --interactive , -i  | 	Befestigen des STDIN eines Containers    |

### docker stop
Stoppen von einen oder mehrere laufende Container.<br>
Der Grundbefehl docker runhat folgende Form:

    docker stop [OPTIONS] CONTAINER [CONTAINER...]

| Name, Kurzform          | Standard      | Beschreibung |
| -------------- | -------------- | -------------- |
| --time , -t    | 10 | Sekunden, um auf Stopp zu warten, bevor der Container gestoppt wird |

**Fazit und Aussicht**, Die Durcharbeitung von 702.1 Container Management gab mir ein besseres Verständnis über die Funktionsweise von Containern.

***

</details>

<details>
<summary>Kapitel: 702.2 Container Deployment and Orchestration</summary>
<br>

# Kapitel: 702.2 Container Deployment and Orchestration (Status: Abgeschlossen)

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

## Architecture and application model of Kubernetes / Definition of Deployments, Services, Pods and ReplicaSets
Kubernetes ist ein von Google entwickeltes Open-Source-Orchestrierungs-Tool zum Verwalten von Microservices oder containerisierten Anwendungen in einem verteilten Cluster von Knoten. Kubernetes bietet hoch belastbare Infrastruktur ohne Ausfallzeiten Bereitstellungsfunktionen, automatisches Rollback, Skalierung und Selbstheilung von Containern (die von Auto-Platzierung besteht, Auto-Neustart, automatische Replikation , und die Skalierung von Containern auf Basis der CPU - Auslastung).
Das Hauptziel von Kubernetes besteht darin, die Komplexität der Verwaltung einer Containerflotte zu verbergen, indem REST-APIs für die erforderlichen Funktionen bereitgestellt werden. Kubernetes ist portabel und kann auf verschiedenen öffentlichen oder privaten Cloud-Plattformen wie AWS, Azure, OpenStack oder Apache Mesos ausgeführt werden. Es kann auch auf Bare-Metal-Maschinen ausgeführt werden.<br>

Kubernetes folgt einer Client-Server-Architektur. Es ist möglich, eine Multi-Master Einrichtung (für hohe Verfügbarkeit) zu haben, aber standardmäßig gibt es einen einzigen Master-Server, der als Kontrollorgan fungiert Knoten und den Punkt des Kontakts. Der Master-Server besteht aus verschiedenen Komponenten, darunter einem Cube-Apiserver, einem etcd-Speicher, einem Cube-Controller-Manager, einem Cloud-Controller-Manager, einem Cube-Scheduler und einem DNS-Server für Kubernetes-Dienste. Zu den Knotenkomponenten gehören kubelet und kube-proxy über Docker.<br>

Im Folgenden sind die Hauptkomponenten aufgeführt, die sich auf dem Masterknoten befinden:

* etcd-Cluster - Ein einfacher, verteilter Schlüsselwertspeicher, in dem die Kubernetes-Clusterdaten (wie Anzahl der Pods, Status, Namespace usw.), API-Objekte und Details zur Serviceerkennung gespeichert werden. Sie ist aus Sicherheitsgründen nur vom API-Server aus zugänglich. etcd ermöglicht die Benachrichtigung des Clusters über Konfigurationsänderungen mithilfe von Beobachtern. Benachrichtigungen sind API-Anforderungen auf jedem etcd-Clusterknoten, um die Aktualisierung von Informationen im Speicher des Knotens auszulösen.
* kube-apiserver - Der Kubernetes API-Server ist die zentrale Verwaltungseinheit, die alle REST-Anforderungen für Änderungen (an Pods, Diensten, Replikationssätzen / Controllern und anderen) empfängt und als Frontend für den Cluster dient. Dies ist auch die einzige Komponente, die mit dem etcd-Cluster kommuniziert und sicherstellt, dass die Daten in etcd gespeichert sind und mit den Servicedetails der bereitgestellten Pods übereinstimmen.
* kube-controller-manager - führt im Hintergrund eine Reihe unterschiedlicher Controller-Prozesse aus (z. B. steuert der Replikations-Controller die Anzahl der Replikate in einem Pod, der Endpunkt-Controller füllt Endpunktobjekte wie Dienste und Pods und andere), um den gemeinsam genutzten Status des zu regeln Cluster und Routineaufgaben ausführen. Wenn eine Änderung in einer Dienstkonfiguration auftritt (z. B. durch Ersetzen des Images, von dem die Pods ausgeführt werden, oder durch Ändern von Parametern in der Konfigurationsdatei), erkennt der Controller die Änderung und beginnt, auf den neuen gewünschten Status hinzuarbeiten.
* Cloud-Controller-Manager - ist für die Verwaltung von Controller-Prozessen mit Abhängigkeiten zum zugrunde liegenden Cloud-Anbieter verantwortlich (falls zutreffend). Wenn ein Controller beispielsweise überprüfen muss, ob ein Knoten beendet wurde, oder Routen, Load Balancer oder Volumes in der Cloud-Infrastruktur einrichten muss, übernimmt der Cloud-Controller-Manager alles.
* kube-scheduler - hilft bei der Planung der Pods (eine Gruppe von Containern, in denen unsere Anwendungsprozesse ausgeführt werden) auf den verschiedenen Knoten basierend auf der Ressourcennutzung. Es liest die Betriebsanforderungen des Dienstes und plant sie auf dem am besten geeigneten Knoten. Wenn die Anwendung beispielsweise 1 GB Arbeitsspeicher und 2 CPU-Kerne benötigt, werden die Pods für diese Anwendung auf einem Knoten mit mindestens diesen Ressourcen geplant. Der Scheduler wird jedes Mal ausgeführt, wenn Pods geplant werden müssen. Der Scheduler muss die insgesamt verfügbaren Ressourcen sowie die Ressourcen kennen, die den vorhandenen Workloads auf jedem Knoten zugeordnet sind.

### Worker komponenten
Nachfolgend sind die Hauptkomponenten eines (Worker-) Knotens aufgeführt:

* kubelet - der Hauptdienst auf einem Knoten, der regelmäßig neue oder geänderte Pod-Spezifikationen (hauptsächlich über den kube-apiserver) erfasst und sicherstellt, dass die Pods und ihre Container in einwandfreiem Zustand sind und im gewünschten Zustand ausgeführt werden. Diese Komponente meldet dem Master auch den Zustand des Hosts, auf dem sie ausgeführt wird.
* kube-proxy - Ein Proxy-Dienst, der auf jedem Arbeitsknoten ausgeführt wird, um einzelne Host-Subnetze zu verarbeiten und Dienste für die Außenwelt bereitzustellen. Es führt eine Anforderungsweiterleitung an die richtigen Pods / Container über die verschiedenen isolierten Netzwerke in einem Cluster durch.

### Kubernetes Konzepte
Um Kubernetes nutzen zu können, muss man die verschiedenen Abstraktionen kennen, mit denen der Status des Systems dargestellt wird, z.B. Dienste, Pods, Volumes, Namespaces und Bereitstellungen.

* Pod - bezieht sich im Allgemeinen auf einen oder mehrere Container, die als einzelne Anwendung gesteuert werden sollen. Ein Pod kapselt Anwendungscontainer, Speicherressourcen, eine eindeutige Netzwerk-ID und andere Informationen zur Ausführung der Container.
* Service- Pods sind flüchtig, d. H. Kubernetes garantiert nicht, dass ein bestimmter physischer Pod am Leben bleibt (z. B. kann der Replikations-Controller einen neuen Satz von Pods beenden und starten). Stattdessen stellt ein Dienst eine logische Gruppe von Pods dar und fungiert als Gateway, sodass (Client-) Pods Anforderungen an den Dienst senden können, ohne nachverfolgen zu müssen, aus welchen physischen Pods der Dienst tatsächlich besteht.
* Volume - Ähnlich wie ein Containervolume in Docker, jedoch gilt ein Kubernetes-Volume für einen gesamten Pod und ist auf allen Containern im Pod aktiviert. Kubernetes garantiert, dass die Daten auch nach einem Neustart des Containers erhalten bleiben. Die Lautstärke wird nur entfernt, wenn der Pod zerstört wird. Außerdem können einem Pod mehrere Volumes (möglicherweise unterschiedlichen Typs) zugeordnet sein.
* Namespace - Ein virtueller Cluster (ein einzelner physischer Cluster kann mehrere virtuelle Cluster ausführen), der für Umgebungen mit vielen Benutzern vorgesehen ist, die auf mehrere Teams oder Projekte verteilt sind, um Bedenken zu isolieren. Ressourcen in einem Namespace müssen eindeutig sein und können nicht auf Ressourcen in einem anderen Namespace zugreifen. Außerdem kann einem Namespace ein Ressourcenkontingent zugewiesen werden, um zu vermeiden, dass mehr als sein Anteil an den Gesamtressourcen des physischen Clusters verbraucht wird.
* Bereitstellung - Beschreibt den gewünschten Status eines Pods oder eines Replikatsatzes in einer Yaml-Datei. Der Deployment Controller aktualisiert dann nach und nach die Umgebung (z. B. Erstellen oder Löschen von Replikaten), bis der aktuelle Status dem in der Deployment-Datei angegebenen gewünschten Status entspricht. Wenn in der yaml-Datei beispielsweise zwei Replikate für einen Pod definiert sind, aber derzeit nur eines ausgeführt wird, wird ein weiteres erstellt. Beachten Sie, dass Replikate, die über eine Bereitstellung verwaltet werden, nicht direkt bearbeitet werden sollten, sondern nur über neue Bereitstellungen.


## Docker-compose
Docker-Compose ist ein Tool zum Definieren und Ausführen von Docker-Anwendungen mit mehreren Containern. Docker-Compose verwendet eine YAML-Datei, um die Dienste Ihrer Anwendung zu konfigurieren. Anschließend erstellen und starten Sie mit einem einzigen Befehl alle Dienste aus Ihrer Konfiguration. Docker-Compose funktioniert in allen Umgebungen: Produktion, Staging, Entwicklung, Test sowie CI-Workflows.<br>

Die Verwendung von Compose besteht im Wesentlichen aus drei Schritten:

* Definieren der Umgebung der App mit einem Dockerfile, damit es überall reproduziert werden kann.
* Definieren der Services, aus denen die App besteht, docker-compose.yml damit die Apps in einer isolierten Umgebung gemeinsam ausgeführt werden können.
* docker-compose up ausführen und Docker-Compose startet und führt die gesamte App aus.

### Beispiel Docker-Compose.yml Datei

    version: '3'
    services:
      web:
        build: .
        ports:
        - "5000:5000"
        volumes:
        - .:/code
        - logvolume01:/var/log
        links:
        - redis
      redis:
        image: redis
    volumes:
      logvolume01: {}

## Docker
Docker vereinfacht die Bereitstellung von Anwendungen, weil sich Container, die alle nötigen Pakete enthalten, leicht als Dateien transportieren und installieren lassen. Container gewährleisten die Trennung und Verwaltung der auf einem Rechner genutzten Ressourcen. Das beinhaltet laut Aussage der Entwickler: 
* Code
* Laufzeitmodul
* Systemwerkzeuge
* Systembibliotheken

Docker basiert auf Linux-Techniken wie Cgroups und Namespaces, um Container zu realisieren. Während anfänglich noch die LXC-Schnittstelle des Linux-Kernels verwendet wurde, haben die Docker-Entwickler mittlerweile eine eigene Programmierschnittstelle namens libcontainer entwickelt, die auch anderen Projekten zur Verfügung steht. Als Speicher-Backend verwendet Docker das Overlay-Dateisystem AuFS, ab Version 0.8 unterstützt die Software aber auch btrfs.
Prinzipiell ist Docker auf die Virtualisierung mit Linux ausgerichtet. Docker kann allerdings auch mittels Hyper-V oder VirtualBox auf Windows und mittels VirtualBox auf macOS verwendet werden. Da die Ressourcentrennung alleine mit den Docker zugrunde liegenden Techniken wie Namespaces und Cgroups nicht völlig sicher ist, hat die Firma Red Hat Unterstützung für die sicherheitsrelevante Kernel-Erweiterung SELinux implementiert, welche die Container auf der Ebene des Host-Systems zusätzlich absichert.


## Kubectl
Der Befehl kubectl ist ein Leitungstool, das mit kube-apiserver interagiert und Befehle an den Masterknoten sendet. Jeder Befehl wird in einen API-Aufruf umgewandelt. Kubectl ist eine Befehlszeilenschnittstelle zum Ausführen von Befehlen für Kubernetes-Cluster. kubectlsucht nach einer Datei namens config im Verzeichnis $ HOME / .kube. Sie können andere kubeconfig- Dateien angeben, indem Sie die Umgebungsvariable KUBECONFIG oder das --kubeconfigFlag setzen.


**Fazit und Aussicht**, Die Durcharbeitung von 701.3 Source Code Management gab mir ein besseres Verständnis darüber was für Features und Eigenschaften Git hat, wie Git diese verwendet und wie man sie selber anwenden.

***

</details>

</details>