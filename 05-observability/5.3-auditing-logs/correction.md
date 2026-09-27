# Correction 5.3

```bash
# 1. Tous les flux vers web
hubble observe --to-pod default/$(kubectl get pod -l app=web -o jsonpath='{.items[0].metadata.name}')
# Ou en ciblant le Service par label
hubble observe --namespace default --to-label app=web

# 2. Flux DROPPED uniquement
hubble observe --namespace default --to-label app=web --verdict DROPPED

# 3. Flux depuis attacker spécifiquement
hubble observe --from-pod default/attacker
# Vérifier que TOUS les flux listés ont un verdict DROPPED, avec une raison du type "Policy denied"

# 4. Résumé attendu (signal suspect vs rejet normal) :
```

**Réponse à la question 4** :
Un simple trafic normalement rejeté (comme dans cet exercice) se traduit par un petit nombre de flux `DROPPED` avec une raison stable et cohérente (`Policy denied`), à un rythme correspondant à une utilisation normale (quelques requêtes). Un **signal suspect** se distinguerait par : (1) un volume anormalement élevé de tentatives en peu de temps depuis une même source (100 requêtes/10s est largement au-dessus d'un usage applicatif normal), (2) potentiellement une variété de paths/méthodes testés (tentative de découverte type scan), et (3) la répétition du même verdict `DROPPED` en boucle rapprochée — ce pattern justifierait une escalade (alerting, blocage IP en amont du cluster) plutôt qu'un simple "c'est normal, la policy fonctionne".

**Point clé pour l'examen** : sache utiliser les filtres `hubble observe` (`--from-pod`, `--to-pod`, `--verdict`, `--protocol`, `--http-status`) de façon combinée pour isoler rapidement un pattern de trafic — c'est plus rapide que de scroller un flux non filtré, et c'est exactement le genre d'efficacité que la contrainte de temps de l'examen récompense.
