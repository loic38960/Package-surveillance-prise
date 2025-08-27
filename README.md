Surveillance des prises Zigbee (Home Assistant)

Surveille automatiquement tes prises Zigbee critiques (congélateur, réfrigérateur, cave, etc.) avec Home Assistant :

⚠️ Alerte si une prise passe off / unavailable / unknown (et reste KO ≥ 5 min).

✅ Rétablissement quand tout redevient OK (≥ 2 min).

🕙 Rapport quotidien à 22:00.

🔁 Vérification périodique toutes les 5 min (détection immédiate si KO).

🔄 Au démarrage de HA : notification même si la prise était déjà KO avant le redémarrage.

Prérequis

Home Assistant (Core / OS / Supervised)

Intégration Zigbee (ZHA ou Zigbee2MQTT)

Un service de notification (mobile, Telegram, Discord…) ou un script de notification

Installation

Activez les packages (si ce n’est pas déjà fait) dans configuration.yaml :

homeassistant:
  packages: !include_dir_named packages


Créez le dossier config/packages/ puis copiez le fichier :

config/packages/surveillance_prises_zigbee.yaml


Redémarrez Home Assistant (ou Vérifier la configuration puis Redémarrer).

Configuration minimale

Dans le fichier, ajoutez vos prises Zigbee dans le groupe :

group:
  prises_zigbee:
    name: "Prises Zigbee surveillées"
    entities:
      - switch.refrigerateur
      - switch.congelateur
      - switch.cav


Par défaut, le package utilise un script pour notifier :

- service: script.notifier_mogo_loic


➡️ Adaptez à votre système de notification :

App mobile : notify.mobile_app_mon_telephone

Telegram : notify.telegram

Discord : notify.discord

Ou gardez votre script existant.

Exemple avec un service mobile :

action:
  - service: notify.mobile_app_mon_telephone
    data:
      title: "⚠️ Prise Zigbee en anomalie"
      message: "⚠️ Congélateur n'est plus alimenté"

Ce que fait chaque automation

Alerte (≥ 5 min)
Déclenche si au moins une prise reste KO pendant 5 min d’affilée.

Rétablissement (≥ 2 min)
Informe quand toutes les prises du groupe sont revenues OK pendant 2 min.

Rapport quotidien (22:00)
Récapitulatif des états, avec la liste des éventuelles anomalies.

Vérification périodique (toutes les 5 min)
Sans temporisation : notifie immédiatement si une prise est KO au moment du check (utile si un événement a été manqué).

Au démarrage de HA
Attend 1 min 30 (modifiable) puis notifie si une prise est KO même si elle l’était déjà avant le reboot.

Options / Personnalisation

⏲️ Changer les horaires/délais

Rapport quotidien : modifiez at: "22:00:00"

Délai d’alerte : for: "00:05:00"

Délai de rétablissement : for: "00:02:00"

Délai au démarrage : delay: "00:01:30" (ou "00:05:00" pour éviter tout faux positif)

🧪 Mode debug (optionnel)
Une notification persistante peut être activée dans l’automatisation #4 pour voir les entités détectées.
Laissez commenté dans la version publique, décommentez pour tester :

# - service: persistent_notification.create
#   data:
#     title: "DEBUG Vérif périodique Zigbee"
#     message: >
#       {% set defauts = expand('group.prises_zigbee')
#          | selectattr('state','in',['off','unavailable','unknown'])
#          | list %}
#       Détections :
#       {% for e in defauts -%}
#       \n• {{ e.name }} ({{ e.entity_id }})
#       {% endfor %}

Dépannage

“Integration 'packages' not found”
→ La clé packages doit être sous homeassistant: (pas à la racine).

Automatisation qui n’apparaît pas / “Jamais”
→ Problème fréquent d’indentation YAML. Toutes les automations doivent être 2 espaces sous automation:.
→ Rechargez les automatisations (Paramètres → Système).

Pas de notif alors que la prise est KO

Vérifiez que l’entité est bien dans le groupe.

Allez dans Outils → Gabarits et testez :

{% for e in expand('group.prises_zigbee') %}
{{ e.entity_id }} → {{ e.state }} (age={{ (as_timestamp(now()) - as_timestamp(e.last_changed))|int }}s)
{% endfor %}


Testez l’automatisation via Exécuter (depuis l’UI) pour valider le service de notification.

Avertissements

Certaines intégrations Zigbee remontent temporairement unavailable au démarrage : d’où le délai au boot (#5).

Pour des équipements critiques (congélateur), privilégiez la fiabilité (ex : prises Zigbee reconnues, routeurs proches, firmware à jour
