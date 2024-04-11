# ESPHome modbus SHT20 Temp & Hum sensor


## ESPHome yaml
```
esphome:
  name: test
  friendly_name: modbus
esp32:
  board: esp32dev
  framework:
    type: arduino
logger:
api:
  encryption:
    key: "zde_napius_vlastni_key_co_ti_vygeneroval_esphome_po_pridani_zaruizeni"
ota:
  password: "zde_napius_vlastni_key_co_ti_vygeneroval_esphome_po_pridani_zaruizeni"
 wifi:
   ssid: !secret wifi_ssid
   password: !secret wifi_password
   ap:
     ssid: "A8M Fallback Hotspot"
     password: "m0gfMavNAM9d"  
 captive_portal:
##############################################

modbus:
  send_wait_time: 200ms
  id: rs485

modbus_controller:
  - id: sht20_1 # senzor sht20 s adresou 1 (0x01) - defaultni
    address: 1
    modbus_id: rs485
    update_interval: 5s
  - id: sht20_4 # senzor sht20 s adresou 4 (0x04) - zmenil jsem u druheho z 1 na 4 (navod na konci)
    address: 0x04
    modbus_id: rs485
    update_interval: 5s

sensor:
## SHT20 - 1  
  - platform: modbus_controller
    modbus_controller_id: sht20_1 # (zde musis dat id tve kontroleru aby esphome vedel kde se ma dotazovat)
    name: "ter_sht1_temp"
    address: 1  # sub adresa registru pro teplotu (zjisteno z datasheetu)
    unit_of_measurement: "°C"
    register_type: read 
    value_type: U_WORD  # 16bitové neznaménkové celé číslo
    accuracy_decimals: 1
    filters:
      - multiply: 0.1  
      - offset: -2.0  # kalibrace
  - platform: modbus_controller
    modbus_controller_id: sht20_1 # (zde musis dat id tve kontroleru aby esphome vedel kde se ma dotazovat)
    name: "ter_sht1_hum"
    address: 2  # subadresa registru pro vlhkost (zjisteno z datasheetu)
    unit_of_measurement: "%"
    register_type: read  
    value_type: U_WORD  # 16bitové neznaménkové celé číslo
    accuracy_decimals: 1
    filters:
      - multiply: 0.1  
      - offset: 0.0  # kalibrace
```

## ESPHome yaml - změna modbus adresy senzoru:
```
esphome:
  name: teresov-a8m
  friendly_name: teresov-a8m

  on_boot:
    priority: -100
    then:
      - lambda: |-
          auto new_address = 15;  // zde nastav novou adresu v hexa (melo by fungovat i v decimalni)

          if(new_address < 1 || new_address > 247) { // Kontrola rozsahu adresy 1-247 (zde nic neupravuj)
            ESP_LOGE("ModbusLambda", "Address needs to be between 1 and 247");
            return;
          }

          esphome::modbus_controller::ModbusController *controller = id(sht20_1); // id musis mit nastaveno podle id tveho modbus_controller zde v programu nize s AKTUALNI adresou zarizeni ktere funguje (default 0x01 = 1)
          auto set_addr_cmd = esphome::modbus_controller::ModbusCommandItem::create_write_single_command(
            controller, 257, new_address);  // Příkaz pro změnu adresy sht20 (pro jiny produkt musis zjistit prepisovaci adresu v datasheetu)

          delay(200);
          controller->queue_command(set_addr_cmd);
          ESP_LOGI("ModbusLambda", "Modbus addres for device changed.");

          // 1. nejdrive musis zjistit aktualni adresu zarizeni a vycitat z nej data aby jsi mel overeno funkcni spojeni
          // 2. prepis new_address na pozadovanou novou adresu 
          // 3. musis nastavit id aktualniho kontroleru na kterem je puvodni funkcni adresa -> controller = id(zde_entitu_modbus_controller) aby program vedel do jakeho zarizeni ma zapsat novou adresu
          // proved update ESP a po dokonceni vypni na 5s ESP z napajeni a opet zapni 
          // po nabootovani by se mela prepsat aktualni adresa na nove pozadovanou 
          // pokud po restartu je zarizeni stale na puvodni adrese tak proved jeste jeden restart
          // adresy muzes zedavat v hexa 0x01 ale i v decimalni soustave 1
          // v pripade problemu over ze mas modbus controller vytvoreny pro starou adresu a i pro novou adresu.. take senzory pro oba aby jsi videl ze uz se data odesilaji z nove adresy
```
