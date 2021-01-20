# Palazzetti-Ha

Just a little reminder of how i use my Stove with Home Assistant. Pretty dirty but does the trick.
I think it will work as long as you use the communication box. You will have to change de ip according to your own network.

sensor.yaml and switch.yaml files are used to have a cleaner configuration.yaml file (i use include: ....yaml)
WIth that done, i've made automations using node red to be notified when the stove have a problem


###### In my sensor.yaml file:
```yaml
  - platform: rest
    resource: http://192.168.1.xxx/cgi-bin/sendmsg.lua?cmd=GET%20TMPS
    name: "temp_poele"
    unit_of_measurement: '°C'
    device_class: temperature
    value_template: '{{ value_json["DATA"]["T1"] }}'

  - platform: rest
    resource: http://192.168.1.xxx/cgi-bin/sendmsg.lua?cmd=GET+STATUS
    name: "id_status_poele"
    value_template: '{{ value_json["DATA"]["STATUS"] }}'

  - platform: rest
    resource: http://192.168.1.xxx/cgi-bin/sendmsg.lua?cmd=GET+STATUS
    name: "id_status_poele_stamp"
    device_class: timestamp
    value_template: '{{ value_json["INFO"]["TS"] }}'
  
Using french below and don't have clue about some error codes but it's not a big deal

  - platform: template
    sensors:
      etat_poele:
        value_template: >- 
          {% if is_state("sensor.id_status_poele", "0") %}
            Eteint
          {% elif is_state("sensor.id_status_poele", "1") %}
            Chargement Pelets
          {% elif is_state("sensor.id_status_poele", "2") %}
            Allumage du Brasier
          {% elif is_state("sensor.id_status_poele", "3") %}
            Fin d'Allumage
          {% elif is_state("sensor.id_status_poele", "4") %}
            Allumé
          {% elif is_state("sensor.id_status_poele", "5") %}
            5?
          {% elif is_state("sensor.id_status_poele", "6") %}
            Cycle de nettoyage
          {% elif is_state("sensor.id_status_poele", "7") %}
            Mode Eco
          {% elif is_state("sensor.id_status_poele", "8") %}
            8?
          {% elif is_state("sensor.id_status_poele", "9") %}
            Alarme Température Fumées
          {% else %}
            Inconnu
          {% endif %}
```
###### In my switch.yaml file:
```yaml
  - platform: command_line
    switches:
      switchpoele:
        command_on: "/usr/bin/curl -X GET http://192.168.1.111/cgi-bin/sendmsg.lua?cmd=CMD+ON"
        command_off: "/usr/bin/curl -X GET http://192.168.1.111/cgi-bin/sendmsg.lua?cmd=CMD+OFF"
        command_state: "/usr/bin/curl -X GET http://192.168.1.111/cgi-bin/sendmsg.lua?cmd=GET+STATUS"
        value_template: >- 
          {% if is_state("sensor.id_status_poele", "1") %}
            True
          {% elif is_state("sensor.id_status_poele", "2") %}
            True
          {% elif is_state("sensor.id_status_poele", "3") %}
            True
          {% elif is_state("sensor.id_status_poele", "4") %}
            True
          {% elif is_state("sensor.id_status_poele", "5") %}
            True
          {% else %}
            False
          {% endif %}
        friendly_name: Switch Poële
```      
######And this is how it appears on my lovelace dashboard:

