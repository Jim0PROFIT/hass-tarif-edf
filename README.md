# Tarif EDF integration for Home Assistant – Fork de Jim0PROFIT

Fork moderne de [delphiki/hass-tarif-edf](https://github.com/delphiki/hass-tarif-edf) avec améliorations Tempo et compatibilité API couleur‑Tempo (`api‑couleur‑tempo.fr`) et open‑dpe.fr, mis à jour pour fonctionner proprement avec Home Assistant récents.

## Changelog & versions

### v2.4.x (Jim0PROFIT fork – actuel)

- **Correction**: `Invalid handler specified` résolu en supprimant l’usage obsolète de `@config_entries.HANDLERS.register` dans `config_flow.py`.
- **Correction**: `CancelledError` et timeout réseaux sur `get_remote_file` corrigés grâce à l’ajout d’un `timeout=10`.
- **API couleur Tempo**: utilisation correcte des endpoints `/api/jourTempo/today` et `/api/jourTempo/tomorrow` au lieu de `/api/jourTempo`.
- **Robustesse JSON**: parsing sécurisé avec `get("codeJour", 0)` et gestion du `CancelledError` dans `coordinator.py`.
- **Sécurisation HTTP**: suppression de `stream=True` inutile, fermeture propre de `response`.
- **Nettoyage code**: `__init__.py` simplifié, `manifest.json` mis à jour vers ton fork (URL documentation/issue_tracker) et version `2.3.3`/`2.4.0` recommandée.

### v2.3.2 (fork FigurinePanda43)

- **Correction : `UnboundLocalError` sur la variable `range`**  
  La variable de boucle `range` dans la gestion des plages HP/HC écrasait le builtin Python, causant un crash à chaque mise à jour du coordinator.

### v2.3.1 – v2.3.0 (FigurinePanda43)

- **Correction : capteurs de prévision toujours indisponibles**  
  Un `KeyError` sur `tempo_variable_hp_ttc` (couleur indéterminée sans cache) faisait planter le coordinator, mettant tous les capteurs en "Indisponible".

- **Correction : données périmées sur `tarif_tempo_couleur`**  
  Entre 06h et 11h, si la couleur du jour était inconnue, le capteur affichait la couleur de la veille au lieu d'être indisponible.

- **Correction : `"indéterminé"` affiché comme valeur**  
  Les capteurs de couleur affichent désormais "Indisponible" (unavailable) plutôt que la chaîne `"indéterminé"` quand la couleur est inconnue.

- **Amélioration : gestion des erreurs réseau**  
  Les appels à l’API couleur sont maintenant protégés par un `try/except` ; une erreur réseau ne fait plus planter le coordinator.

- **Amélioration : cache des prévisions**  
  L'API de prévision open‑dpe.fr n'est plus appelée chaque minute mais toutes les heures, avec fallback sur le dernier résultat connu.

- **Nouveaux capteurs : prévisions Tempo J+1 à J+9**  
  Ajout de capteurs affichant la couleur prédite et la probabilité pour les 9 prochains jours (source: open‑dpe.fr).

- **Correction du bug "indéterminé" 00h‑11h**  
  La couleur d'aujourd'hui est résolue depuis le cache (couleur annoncée la veille), `tarif_tempo_couleur` continue d'utiliser la couleur d'hier jusqu'à 06h.

### v2.2.1 – v2.2.0 (FigurinePanda43)

- **Correction du fuseau horaire**  
  Le changement de jour Tempo respecte maintenant le fuseau horaire configuré dans Home Assistant (et non plus UTC).

- **Persistance du cache Tempo**  
  La couleur de "demain" connue la veille est sauvegardée sur disque et survit aux redémarrages de Home Assistant.

- **Correction du bug "indéterminé"**  
  Après minuit, si l'API retourne "indéterminé" pour aujourd'hui, l'intégration réutilise la couleur connue la veille comme couleur de "demain".

---

## Installation

### Via HACS (dépôt personnalisé)

1. Ouvrez HACS depuis l’interface Home Assistant.  
2. Cliquez sur `…` en haut à droite → **Dépôts personnalisés**.  
3. Ajoutez l’URL de **ton fork** :  
   - `https://github.com/Jim0PROFIT/hass-tarif-edf`  
4. Cliquez sur **Ajouter**.  
5. Cliquez sur **Intégrations** dans HACS.  
6. Recherchez **Tarif EDF** et installez‑la.

### Installation manuelle

Copiez le dossier `tarif_edf` dans `custom_components` de votre configuration Home Assistant.

```bash
cd /chemin/vers/votre/config/custom_components/
# Télécharge ton fork (ou ton zip GitHub)
wget https://github.com/Jim0PROFIT/hass-tarif-edf/archive/refs/heads/main.zip
unzip main.zip
mv hass-tarif-edf-main/tarif_edf .
rm -rf hass-tarif-edf-main main.zip
```

Puis **redémarre Home Assistant**.

---

## Configuration

[![Open Home Assistant config flow](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=tarif_edf)

### Paramètres principaux

| Paramètre        | Description                                      | Valeurs possibles |
|------------------|--------------------------------------------------|-------------------|
| `contract_power` | Puissance souscrite de ton contrat               | 3, 6, 9, 12, 15, 18, 30, 36 kVA |
| `contract_type`  | Type de contrat EDF                              | Base, Heures pleines / Heures creuses, Tempo |

### Options (post‑installation)

| Option                    | Description                                                     | Valeur par défaut |
|---------------------------|-----------------------------------------------------------------|-------------------|
| `refresh_interval`        | Intervalle de rafraîchissement des tarifs (jours)              | 1 |
| `off_peak_hours_ranges`   | Plages horaires Horaire Creuses, format `HH:MM-HH:MM`          | 22:00-06:00 (Tempo) |

**Format des plages horaires** :  
`HH:MM-HH:MM` séparées par des virgules.  
Exemple :  
- `22:00-06:00`  
- `01:30-07:30,12:30-14:30`

---

## Capteurs disponibles

### Capteurs communs (tous contrats)

| Capteur                                                  | Description                                   | Unité       |
|----------------------------------------------------------|-----------------------------------------------|-------------|
| `sensor.puissance_souscrite_[type]_[power]kva`          | Puissance souscrite                           | kVA         |
| `sensor.tarif_actuel_[type]_[power]kva_ttc`              | Tarif TTC actuellement appliqué               | EUR/kWh     |

### Contrat Base

| Capteur                         | Description                    | Unité |
|---------------------------------|--------------------------------|-------|
| `sensor.tarif_base_ttc`         | Tarif de base                  | EUR/kWh |

### Contrat HP/HC

| Capteur                                   | Description                     | Unité |
|-------------------------------------------|---------------------------------|-------|
| `sensor.tarif_heures_creuses_ttc`         | Tarif heures creuses            | EUR/kWh |
| `sensor.tarif_heures_pleines_ttc`         | Tarif heures pleines            | EUR/kWh |

### Contrat Tempo

| Capteur                                       | Description | Unité |
|-----------------------------------------------|------------|-------|
| `sensor.tarif_tempo_couleur`                  | Couleur Tempo active pour la facturation (changée à 06:00, basée sur HP/HC) | - |
| `sensor.tarif_tempo_couleur_hier`             | Couleur Tempo d’hier | - |
| `sensor.tarif_tempo_couleur_aujourd_hui`      | Couleur Tempo du jour (00:00 → 00:00) | - |
| `sensor.tarif_tempo_couleur_demain`           | Couleur Tempo de demain (disponible après 11:00) | - |
| `sensor.tarif_tempo_heures_creuses_ttc`       | Tarif Hors‑Pique actuel | EUR/kWh |
| `sensor.tarif_tempo_heures_pleines_ttc`       | Tarif Pique actuel | EUR/kWh |
| `sensor.tarif_bleu_tempo_heures_creuses_ttc`  | Tarif HP Tempo bleu | EUR/kWh |
| `sensor.tarif_bleu_tempo_heures_pleines_ttc`  | Tarif HC Tempo bleu | EUR/kWh |
| `sensor.tarif_blanc_tempo_heures_creuses_ttc` | Tarif HP Tempo blanc | EUR/kWh |
| `sensor.tarif_blanc_tempo_heures_pleines_ttc` | Tarif HC Tempo blanc | EUR/kWh |
| `sensor.tarif_rouge_tempo_heures_creuses_ttc` | Tarif HP Tempo rouge | EUR/kWh |
| `sensor.tarif_rouge_tempo_heures_pleines_ttc` | Tarif HC Tempo rouge | EUR/kWh |

### Prévisions Tempo (J+1 à J+9)

| Capteur                         | Description | Attributs |
|---------------------------------|-------------|-----------|
| `sensor.tempo_prevision_j_1`    | Prévision J+1 | `probabilite`, `date` |
| `sensor.tempo_prevision_j_2`    | Prévision J+2 | `probabilite`, `date` |
| `sensor.tempo_prevision_j_3`    | Prévision J+3 | `probabilite`, `date` |
| `sensor.tempo_prevision_j_4`    | Prévision J+4 | `probabilite`, `date` |
| `sensor.tempo_prevision_j_5`    | Prévision J+5 | `probabilite`, `date` |
| `sensor.tempo_prevision_j_6`    | Prévision J+6 | `probabilite`, `date` |
| `sensor.tempo_prevision_j_7`    | Prévision J+7 | `probabilite`, `date` |
| `sensor.tempo_prevision_j_8`    | Prévision J+8 | `probabilite`, `date` |
| `sensor.tempo_prevision_j_9`    | Prévision J+9 | `probabilite`, `date` |

**Attributs courants** :
- `probabilite` : 0–100 (confiance)
- `probabilite_pourcent` : format `"XX%"`
- `date` : `YYYY-MM-DD`
- `jour` : J+N

---

## Fonctionnement du Tempo – schéma rapide

### Couleurs Tempo

- **Bleu** : jours les moins chers (≈300 jours/an)  
- **Blanc** : intermédiaire (≈43 jours/an)  
- **Rouge** : plus chers (≈22 jours/an) [web:1]

### Jour, heures creuses / pleines

- **Jour Tempo** : 06:00 → 06:00 le lendemain  
- **Heures creuses (HC)** : 22:00 → 06:00  
- **Heures pleines (HP)** : 06:00 → 22:00  
- **Couleur de demain disponible** : disponible à partir de 11:00 sur l’API couleur Tempo. [web:31]

### Différence entre `tarif_tempo_couleur` et `tarif_tempo_couleur_aujourd_hui`

|                          | `tarif_tempo_couleur_aujourd_hui` | `tarif_tempo_couleur` |
|--------------------------|-------------------------------------|------------------------|
| Change à                 | 00:00 (minuit)                     | 06:00 |
| Représente               | jour calendrier                      | jour facturation EDF |
| 00:00–06:00              | couleur d’aujourd’hui               | couleur d’hier |
| Après 06:00              | couleur d’aujourd’hui               | couleur d’aujourd’hui |

C’est donc **normal** que les deux capteurs diffèrent entre 00:00 et 06:00.

### Gestion du cache

- La couleur de demain annoncée par EDF (après 11:00) est sauvegardée sur disque.  
- Si HA redémarre après minuit et avant 11h, l’intégration réutilise automatiquement cette couleur pour `tarif_tempo_couleur_aujourd_hui`.  
- Si aucun cache n’est disponible (première installation, suppression du cache), les capteurs de couleur passent en **Indisponible** jusqu’à obtention d’une couleur valide.

---

## Sources de données

- **Tarifs EDF** : données officielles publiées sur [data.gouv.fr](https://www.data.gouv.fr/)  
- **Couleurs Tempo (jour/hier/demain)** : [api‑couleur‑tempo.fr](https://www.api-couleur-tempo.fr/)  
- **Prévisions Tempo (J+1 à J+9)** : [open‑dpe.fr – Tempo Forecast](https://open-dpe.fr/tempo-forecast/)

---

## Maintien de ce fork

Ce fork est maintenu par **Jim0PROFIT** et vise à :
- corriger `CancelledError` / timeout réseau,
- utiliser correctement les endpoints modernes de `api‑couleur‑tempo.fr`,
- fixer `Invalid handler specified` sur les versions récentes de Home Assistant,
- et garder toutes les améliorations de FigurinePanda43 et delphiki.

Les PR sont bienvenues. Pour signaler un bug ou une amélioration, utilise l’[issue tracker](https://github.com/Jim0PROFIT/hass-tarif-edf/issues) de ce dépôt.

---

## License

Cette intégration est basée sur le code original de [delphiki/hass-tarif-edf](https://github.com/delphiki/hass-tarif-edf) et conserve la même licence.
