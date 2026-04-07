#  Load Balancer HAProxy — Weighted Round Robin

Mise en place d'un load balancer HAProxy avec la méthode **Weighted Round Robin** dans le cadre de ma formation à l'IPSSI Lille (2025/2026).

## 🏗️ Architecture

```
                    ┌─────────────────┐
                    │   CLIENT HTTP   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │    HA-PROXY     │
                    │ 192.168.10.254  │
                    │   (Port 80)     │
                    └───┬─────────┬───┘
                        │         │
           75% du trafic│         │25% du trafic
           (weight 3)   │         │  (weight 1)
                ┌───────▼──┐  ┌───▼───────┐
                │ srv-web1 │  │ srv-web2  │
                │10.10.10.10│  │10.10.10.20│
                └──────────┘  └───────────┘
```

##  Plan d'adressage

| Machine | Interface | Adresse IP |
|---------|-----------|------------|
| HA-Proxy | Front-end | 192.168.10.254/24 |
| HA-Proxy | Back-end | 10.10.10.1/24 |
| srv-web1 | Back-end | 10.10.10.10/24 |
| srv-web2 | Back-end | 10.10.10.20/24 |

##  Configuration HAProxy

Le fichier `haproxy.cfg` configure :
- **Frontend** : écoute sur le port 80
- **Backend** : Weighted Round Robin avec health checks HTTP
- **Stats** : interface de supervision sur le port 8080

### Principe du Weighted Round Robin
- `srv-web1` : **poids 3** → reçoit **75%** du trafic
- `srv-web2` : **poids 1** → reçoit **25%** du trafic

##  Déploiement

```bash
# 1. Installer HAProxy sur Debian
sudo apt update && sudo apt install haproxy -y

# 2. Copier la configuration
sudo cp haproxy.cfg /etc/haproxy/haproxy.cfg

# 3. Valider la configuration
sudo haproxy -c -f /etc/haproxy/haproxy.cfg

# 4. Redémarrer le service
sudo systemctl restart haproxy
sudo systemctl enable haproxy

# 5. Vérifier le statut
sudo systemctl status haproxy
```

##  Test du Weighted Round Robin

```bash
# Envoyer 4 requêtes successives
for i in 1 2 3 4; do
    curl http://192.168.10.254
done

# Résultat attendu :
# serveur web 1
# serveur web 1
# serveur web 1
# serveur web 2
```

##  Interface de statistiques

Accessible via : `http://192.168.10.254:8080/stats`  
Login : `ha-admin` / Mot de passe : `Azerty/123`

## 👨‍💻 Auteur

**Fritzel ADJOVI** — IPSSI Lille 2025/2026  
[LinkedIn](https://www.linkedin.com/in/fritzel-adjovi-a95203386) · [GitHub](https://github.com/fritzel-adjovi)
