# Aufgabe 1
### Schritt 1: Vorbereitung der EC2-Instanzen

1. **EC2-Instanzen starten:**
   - Ich starte drei EC2-Instanzen (EC2-1, EC2-2 und EC2-3) und stelle sicher, dass sie über ausreichende Ressourcen verfügen und sich in derselben VPC befinden.

2. **SSH-Zugang einrichten:**
   - Ich stelle sicher, dass ich über SSH-Zugang zu allen Instanzen verfüge. Ich nutze dazu den entsprechenden SSH-Schlüssel.

3. **Updates und Abhängigkeiten installieren:**
   - Auf jeder Instanz verbinde ich mich per SSH und führe die folgenden Befehle aus, um das System zu aktualisieren und notwendige Tools zu installieren:
     ```bash
     sudo apt-get update
     sudo apt-get install -y curl
     ```

### Schritt 2: K3s auf dem Master-Node installieren

1. **K3s auf dem Master-Node (EC2-1) installieren:**
   - Ich verbinde mich mit dem ersten EC2-Server (Master-Node) und installiere K3s:
     ```bash
     curl -sfL https://get.k3s.io | sh -
     ```

2. **Kubeconfig-Datei kopieren:**
   - Um später auf den Cluster zugreifen zu können, kopiere ich die Kubeconfig-Datei:
     ```bash
     mkdir -p ~/.kube
     sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
     sudo chown $(id -u):$(id -g) ~/.kube/config
     ```

3. **Cluster überprüfen:**
   - Ich überprüfe, ob der Master-Node richtig läuft:
     ```bash
     kubectl get nodes
     ```

### Schritt 3: K3s auf den Worker-Nodes installieren

1. **K3s Token auf dem Master-Node abrufen:**
   - Ich sichere das Token auf dem Master-Node, um es später zu verwenden:
     ```bash
     sudo cat /var/lib/rancher/k3s/server/node-token
     ```

2. **K3s auf den Worker-Nodes (EC2-2 und EC2-3) installieren:**
   - Auf den Worker-Nodes verbinde ich mich per SSH und installiere K3s mit dem folgenden Befehl:
     ```bash
     curl -sfL https://get.k3s.io | K3S_URL=https://<MASTER_NODE_IP>:6443 K3S_TOKEN=<NODE_TOKEN> sh -
     ```
   - Ich ersetze `<MASTER_NODE_IP>` durch die IP-Adresse des Master-Nodes und `<NODE_TOKEN>` durch das Token, das ich zuvor gesichert habe.

3. **Cluster überprüfen:**
   - Ich gehe zurück zum Master-Node und überprüfe, ob alle Nodes dem Cluster beigetreten sind:
     ```bash
     kubectl get nodes
     ```

### Schritt 4: Abschluss

1. **Testen des Clusters:**
   - Ich stelle sicher, dass mein K3s-Cluster wie erwartet funktioniert, indem ich einen einfachen Pod deploye:
     ```bash
     kubectl run hello-world --image=nginx --restart=Never
     kubectl get pods
     ```

Diese Schritte sind direkt aus der [K3s Quick-Start Anleitung](https://docs.k3s.io/quick-start) abgeleitet und angepasst an die Verwendung von EC2-Instanzen.