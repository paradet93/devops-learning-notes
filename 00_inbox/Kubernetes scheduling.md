## 1. The ELI5 (Erklärung für 5-Jährige)
*(Hier kommt deine Analogie rein - z.B. Tetris)*

## 2. Der Ablauf (Wie funktioniert es?)
 sequenceDiagram
    User->>API Server: Ich will einen Pod!
    API Server->>etcd: Schreib das auf (Pending)
    Scheduler->>API Server: Oh, ein neuer Pod ohne Node?
    Scheduler->>Scheduler: Ich rechne... Node 3 passt!
    Scheduler->>API Server: Pack ihn auf Node 3.
    Kubelet (Node 3)->>API Server: Ah, Arbeit für mich!
    Kubelet (Node 3)->>Docker: Start den Container!

## 3. Warum brauchen wir das?
*(Welches Problem löst das? Was war "früher" ohne das schlechter?)*

## 4. Aha-Momente / Überraschungen
*(Dinge, die ich falsch dachte oder die nicht intuitiv waren)*