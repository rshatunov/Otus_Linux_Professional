# Lab-05
## Работа с LVM
### Цели:
- Научиться создавать и работать с логическими томами.

 <details>
<summary>  Уменьшить том под / до 8G: </summary>

```
#### Базовая настройка ####
set system host-name Spine-01
delete interfaces
set interfaces xe-0/0/1 description "### Link to Leaf-01 int xe-0/0/1 ###"
set interfaces xe-0/0/1.0 family inet address 10.2.1.0/31
set interfaces xe-0/0/2 description "### Link to Leaf-02 int xe-0/0/1 ###"
set interfaces xe-0/0/2.0 family inet address 10.2.1.2/31
set interfaces xe-0/0/3 description "### Link to Leaf-03 int xe-0/0/1 ###"
set interfaces xe-0/0/3.0 family inet address 10.2.1.4/31
set interfaces em1 description "### Link to vQFX-PFE int em1 ###"
set interfaces em1.0 family inet address 169.254.0.2/24
set interfaces lo0.0 family inet address 10.0.1.0/32

#### Настройка IS-IS ####
set interfaces lo0.0 family iso address 49.0001.0100.0000.1000.00
set interfaces xe-0/0/1.0 family iso
set interfaces xe-0/0/2.0 family iso
set interfaces xe-0/0/3.0 family iso
set protocols isis level 2 wide-metrics-only
set protocols isis level 1 disable
set protocols isis interface xe-0/0/1.0 point-to-point
set protocols isis interface xe-0/0/1.0 family inet bfd-liveness-detection minimum-interval 200 multiplier 3
set protocols isis interface xe-0/0/1.0 level 2 metric 10
set protocols isis interface xe-0/0/1.0 level 2 hello-authentication-key "$9$5Q/tIRSleW36evWX-d5Qz6tu"
set protocols isis interface xe-0/0/1.0 level 2 hello-authentication-type md5
set protocols isis interface xe-0/0/2.0 point-to-point
set protocols isis interface xe-0/0/2.0 family inet bfd-liveness-detection minimum-interval 200 multiplier 3
set protocols isis interface xe-0/0/2.0 level 2 metric 10
set protocols isis interface xe-0/0/2.0 level 2 hello-authentication-key "$9$5Q/tIRSleW36evWX-d5Qz6tu"
set protocols isis interface xe-0/0/2.0 level 2 hello-authentication-type md5
set protocols isis interface xe-0/0/3.0 point-to-point
set protocols isis interface xe-0/0/3.0 family inet bfd-liveness-detection minimum-interval 200 multiplier 3
set protocols isis interface xe-0/0/3.0 level 2 metric 10
set protocols isis interface xe-0/0/3.0 level 2 hello-authentication-key "$9$5Q/tIRSleW36evWX-d5Qz6tu"
set protocols isis interface xe-0/0/3.0 level 2 hello-authentication-type md5
set protocols isis interface interface lo0.0 passive
```
</details>

