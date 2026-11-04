# Exercice 1 — Adresses IP, adresses MAC et ping

**Niveau :** ⭐ · **Durée :** ~45 min
**📖 Fiche théorique à lire avant :** `https://<votre-site>.pythonanywhere.com/bases`

## 🎯 Objectifs

- Lire l'adresse IP, le masque et l'adresse MAC d'une machine.
- Comprendre le rôle de la **passerelle** (le routeur).
- Voir **ARP** remplir la table des voisins.
- Tester la connectivité avec `ping` et `traceroute`.

> 📍 Tout cet exercice se fait dans le **terminal du client** 💻.
> 🖥️ Si besoin : `cd /workspaces/*/lab && docker compose up -d && docker exec -it lab_client bash`

---

## Partie A — Qui suis-je ?

### Étape 1 — Mes adresses

💻 Affichez vos interfaces réseau :

```bash
ip addr
```

👀 Repérez l'interface **`eth0`**. Sur la ligne `link/ether` se trouve l'**adresse MAC** ;
sur la ligne `inet` se trouvent l'**adresse IP** et le **masque** (après le `/`).

> 💡 L'interface `lo` (*loopback*, `127.0.0.1`) est une interface virtuelle qui permet à la
> machine de se parler à elle-même. On l'ignore dans cet exercice.

**Question A.1.** Complétez le tableau pour le client :

| Information        | Valeur pour le client |
| ------------------ | --------------------- |
| Nom de l'interface |        eth0           |
| Adresse MAC        |   66:cb:04:bb:eb:e9   |
| Adresse IP         |     172.20.1.50       |
| Masque (notation `/`) |      /24           |
| Adresse du réseau  |     172.20.1.0        |

**Question A.2.** Le masque est `/24`. Combien de machines différentes peut-on placer dans ce réseau ?
(Indice : le dernier nombre de l'adresse peut varier de 0 à 255, mais 2 valeurs sont réservées.)

> 💬 **Votre réponse :**
>
> 254 machines : 256 adresses possibles, moins l'adresse du réseau (.0) et l'adresse de broadcast (.255).

### Étape 2 — Ma passerelle

💻 Affichez la table de routage :

```bash
ip route
```

👀 Deux lignes apparaissent :

- `172.20.1.0/24 dev eth0 …` → « pour joindre mon réseau local, j'envoie directement sur `eth0` » ;
- `default via 172.20.1.254 …` → « pour **tout le reste**, je passe par `172.20.1.254` ».

**Question A.3.** Quelle est l'adresse de la passerelle du client ? À quelle machine du lab correspond-elle ?

> 💬 **Votre réponse :**
>
> 172.20.1.254, c'est le routeur NAT (nat-router).

---

## Partie B — Mes voisins (ARP)

### Étape 3 — Table ARP vide

💻 Videz puis affichez la table ARP (la liste des voisins dont le client connaît la MAC) :

```bash
ip neigh flush all
ip neigh
```

👀 La table est vide (ou ne contient que des lignes `FAILED`).

### Étape 4 — Un ping fait apparaître un voisin

💻 Envoyez 3 pings au serveur DHCP, qui est sur le **même réseau** que vous :

```bash
ping -c 3 172.20.1.2
```

👀 Vous devez voir 3 lignes `64 bytes from 172.20.1.2: icmp_seq=… ttl=64 time=… ms`
puis `0% packet loss`.

💻 Affichez à nouveau la table ARP :

```bash
ip neigh
```

**Question B.1.** Quelle nouvelle ligne est apparue ? Que contient-elle ? Pourquoi le client a-t-il
eu besoin de cette information **avant** de pouvoir envoyer le ping ?

> 💬 **Votre réponse :**
>
> Une entrée ARP est apparue : 172.20.1.2 avec la MAC 02:22:60:e8:f8:8e. Le client avait besoin de cette MAC pour construire la trame Ethernet avant d'envoyer le ping.

### Étape 5 — Joindre une machine d'un autre réseau

💻 Videz la table ARP, puis faites un ping vers le site web `internet`, qui est sur un **autre réseau** :

```bash
ip neigh flush all
ping -c 3 172.20.0.10
ip neigh
```

**Question B.2.** Cette fois, le client n'a **pas** appris la MAC de `172.20.0.10`. De quelle IP
a-t-il appris la MAC ? Expliquez pourquoi, en vous aidant de la fiche théorique.

> 💬 **Votre réponse :**
>
> De 172.20.1.254 (le routeur NAT). Le site est sur un autre réseau, donc le client passe par sa passerelle.

---

## Partie C — Le chemin vers Internet

### Étape 6 — Comparer les TTL

💻 Faites un ping vers le routeur, puis vers le site web, et regardez la valeur `ttl=` :

```bash
ping -c 2 172.20.1.254
ping -c 2 172.20.0.10
```

**Question C.1.** Notez le TTL des réponses dans les deux cas. Sont-ils identiques ? Que signifie
cette différence ? (Indice : chaque routeur traversé diminue le TTL de 1.)

> 💬 **Votre réponse :**
>
> Non : TTL 64 vers la passerelle, 63 vers le site. Le paquet vers le site a traversé un routeur (le NAT), qui a retiré 1 au TTL.

### Étape 7 — traceroute

💻 Affichez la liste des routeurs traversés pour atteindre le site web :

```bash
traceroute -n 172.20.0.10
```

👀 Chaque ligne numérotée est un **saut** (*hop*).

**Question C.2.** Combien de sauts y a-t-il ? Quelle est l'adresse du premier saut ?
Dessinez (en texte) le chemin suivi : `client → … → internet`.

> 💬 **Votre réponse :**
>
> Il y a 2 sauts. Le premier saut est 172.20.1.254, c'est le routeur NAT (la passerelle du client). Le deuxième saut est 172.20.0.10, c'est le site web lui-même. Chemin : client (172.20.1.50) → nat-router (172.20.1.254) → internet (172.20.0.10)

### Étape 8 — Le lab est isolé

💻 Essayez de joindre un vrai serveur d'Internet (le DNS de Google) :

```bash
ping -c 2 -W 2 8.8.8.8
```

**Question C.3.** Que se passe-t-il ? Pourquoi, selon vous ? (Pensez à ce qu'il y a derrière le routeur du lab.)

> 💬 **Votre réponse :**
>
> Le ping échoue : le routeur NAT (172.20.1.254) répond « Destination Net Unreachable ». Le lab est isolé : derrière le routeur, il n'y a que le réseau du site web (172.20.0.0/24), aucune route vers le vrai Internet. Le routeur ne sait donc pas où envoyer les paquets pour 8.8.8.8.

---

## Partie D — Synthèse

**Question D.1.** Complétez avec vos propres mots :

| Notion        | Explication en une phrase |
| ------------- | ------------------------- |
| Adresse IP    |C'est l'adresse d'une machine sur le réseau.|
| Adresse MAC   |C'est l'identifiant de la carte réseau qui est unique.|
| Passerelle    |C'est un outil matériel ou logiciel qui relie deux réseaux différents.|
| ARP           |C'est le mécanisme qui permet de convertir une adresse IP en adresse MAC.|
| TTL           |C'est une valeur numérique qui indique combien de routeurs un paquet peut transiter.|

**Question D.2.** Vrai ou faux ? Justifiez : « Pour envoyer un paquet à `172.20.0.10`, le client met
dans la trame la MAC de `172.20.0.10`. »

> 💬 **Votre réponse :**
>
> Faux. 172.20.0.10 est sur un autre réseau, donc le client met la MAC de sa passerelle (172.20.1.254). On l'a vu avec ip neigh : seule la MAC du routeur apparaissait.

---

## ✅ Ce qu'il faut retenir

- `ip addr` → mes adresses ; `ip route` → ma passerelle ; `ip neigh` → mes voisins (ARP).
- Sur le **même réseau**, on parle directement à la machine (on demande sa MAC avec ARP).
- Vers un **autre réseau**, on confie le paquet à la **passerelle** : la MAC utilisée est celle du routeur.
- Le **TTL** diminue de 1 à chaque routeur traversé.

## 🆘 En cas de problème

| Problème | Solution |
| -------- | -------- |
| `ping: connect: Network is unreachable` | Le client n'a plus d'IP ou de route. 🖥️ `docker compose restart client` |
| `ip neigh flush all` affiche une erreur | Sans gravité, continuez |
| `traceroute` affiche des `* * *` | Attendez la fin, ou ajoutez `-w 1` pour accélérer |
