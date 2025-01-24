

### **2. Script TCL pour la simulation (Exemple avec GPSR)**

Voici un script **TCL** typique pour simuler un réseau sans fil avec le protocole GPSR dans NS2 :

```tcl
# Création de la simulation
set ns [new Simulator]

# Activation des fichiers de trace
set tracefile [open gpsr_trace.tr w]
set namfile [open gpsr_simulation.nam w]
$ns trace-all $tracefile
$ns namtrace-all $namfile

# Définir les paramètres du réseau
set val(chan)         Channel/WirelessChannel
set val(prop)         Propagation/TwoRayGround
set val(netif)        Phy/WirelessPhy
set val(mac)          Mac/802_11
set val(ifq)          Queue/DropTail/PriQueue
set val(ll)           LL
set val(ant)          Antenna/OmniAntenna
set val(x)            1000     ;# Dimension x
set val(y)            1000     ;# Dimension y
set val(nn)           10       ;# Nombre de nœuds
set val(stop)         50       ;# Temps de fin (en secondes)

# Création du canal sans fil
set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)

create-god $val(nn)

# Configuration des nœuds
$ns node-config -adhocRouting GPSR \
                -llType $val(ll) \
                -macType $val(mac) \
                -ifqType $val(ifq) \
                -antType $val(ant) \
                -propType $val(prop) \
                -phyType $val(netif) \
                -channelType $val(chan) \
                -topoInstance $topo \
                -agentTrace ON \
                -routerTrace ON \
                -macTrace ON

# Création des nœuds
for {set i 0} {$i < $val(nn)} {incr i} {
    set node($i) [$ns node]
    $node($i) random-motion 0 ;# Désactiver le mouvement aléatoire
}

# Configuration du trafic (CBR entre nœuds)
set udp [new Agent/UDP]
set cbr [new Application/Traffic/CBR]
$cbr attach-agent $udp
$ns attach-agent $node(0) $udp

set sink [new Agent/Null]
$ns attach-agent $node(9) $sink
$ns connect $udp $sink
$cbr set rate_ 1Mb
$ns at 1.0 "$cbr start"

# Définir le mouvement des nœuds
$node(0) set X_ 100.0
$node(0) set Y_ 200.0
$node(0) set Z_ 0.0

$node(9) set X_ 800.0
$node(9) set Y_ 900.0
$node(9) set Z_ 0.0

# Lancer la simulation
$ns at $val(stop) "stop"
$ns at $val(stop) "exit"
$ns run
```

---

### **3. Script AWK pour analyser les résultats**

Exemple de script AWK pour calculer le débit (throughput) :

```awk
# awk script: throughput.awk
BEGIN {
    sent = 0
    received = 0
}

{
    if ($1 == "s" && $4 == "AGT") {
        sent++
    }
    if ($1 == "r" && $4 == "AGT") {
        received++
    }
}

END {
    print "Paquets envoyés : " sent
    print "Paquets reçus : " received
    print "Taux de réception : " (received / sent) * 100 "%"
}
```

---

### **4. Fichiers de trace et résultats**
#### Exemple de contenu d’un fichier trace (`gpsr_trace.tr`) :
```plaintext
s 0.500000000 _0_ AGT --- 0 gpsr 512 [0 0 0 0]
r 0.505000000 _9_ AGT --- 0 gpsr 512 [0 0 0 0]
```

#### Commande pour analyser le fichier trace :
```bash
awk -f throughput.awk gpsr_trace.tr > gpsr_results.txt
```

#### Contenu du fichier résultats (`gpsr_results.txt`) :
```plaintext
Paquets envoyés : 1
Paquets reçus : 1
Taux de réception : 100%
```

---

### **5. Commandes Linux utilisées**
1. **Lancer la simulation NS2** :
   ```bash
   ns gpsr_simulation.tcl
   ```
2. **Analyser le fichier de trace avec AWK** :
   ```bash
   awk -f throughput.awk gpsr_trace.tr
   ```
3. **Visualiser le fichier NAM** :
   ```bash
   nam gpsr_simulation.nam
   ```

Avec ces étapes, vous serez en mesure de couvrir tous les aspects demandés pour votre exposé ! Si vous avez besoin de clarifications, faites-le-moi savoir.
