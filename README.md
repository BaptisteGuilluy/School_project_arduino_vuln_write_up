
# School_project_arduino_vuln_write_up
 

## SUMMARY
 
1. [Hardware](#hardware)
2. [Asset description](#asset-description)
3. [Attack tree](#attack-tree)
4. [Vulnerability severity assessment](#vulnerability-severity-assessment)
5. [Attack(s) description and implementation](#attacks-description-and-implementation)
6. [Proposed counter-measure](#Proposed-counter-measures)
7. [References](#references)
   
---
 
## Hardware
 
### Target 
 
| Elements | Specifications |
|---|---|
| Board | Arduino UNO |
| µC |  ATmega328P |
| Entry point | UART, USB |
 
### Tools used
 
| Tool  | Goal | Software |
|---|---|---|
| Logic analyzer | Measure all informations transit between the exterior and the interior of the board | Pulseview |
| Chipwhisperer board CWNANO edition | Specialized board to try multiples hardware attack (in this case I've used CPA) | Chipwhisperer Software suite |
 
### Montage
 
<img width="947" height="623" alt="image" src="https://github.com/user-attachments/assets/c6e742ed-ce80-40ea-b748-0fdbfd3ac194" />

 le montage au dessus permet plusieurs choses. En premier lieu, cela permet de déplacer le µC vers la breadboard afin de pouvoir se connecter directement au pin 16 et 17 qui agiront ici comme RX et TX. On peut utiliser la pin 7 comme pin de mesure (la pin au milieu de l'ilot des 3 pins ou measure est marqué devant) pour le module chipwhisperer en y branchant un shunt pour y observer la mesure de puissance. La pin 19 du µC permettra de générer le signaux trigger nécessaire à la pin 16 du module chipwhisperer. La pin 16 du module est dédié au entré trigger et la pin 19 correspond en fait à la pin 13 de l'arduino uno, determiné via l'analyseur logique (C'est celui-ci qui nous aura permis de determiner les pins 16 et 17 quant à la connexion UART). Ces pins n'ont pas été modifié afin d'éviter de générer des problèmes à cause du code. le reste du montage est nécessaire afin d'alimenter les différents modules et les valeurs de capacités ont été déterminés via ce github : https://github.com/parastuffs/Embedded-Security/wiki/Connecting-to-an-external-target . 

 Ce montage ne sera pas modifié durant l'exploit car celui-ci sera entièrement suffisant.
 
 
---
 
## Asset description

| Asset | Location | Confidentiality |
|---|---|---|
|key entry location | in one of the entry point | medium as limited by the number of interface |
| Keys | in the µC | high |

The key will let us get into the main architecture. In theory the key hold the most value, nonetheless if the firmware is known and not carefully hash we can use LLMS or other tools to decrypt it making all this attack meaningless.
 
---
 
## Attack tree
 
GOAL : **This write-up is educational so the goal is just to get into the arduino and obtain one salt and one hash.**
 
```mermaid
graph TD
    A[goal : get the secret] --> B[Side channel attack]
    B --> C1[Comsommation analysis]
    C1 --> C1a[SPA]
    C1a -. Hashing of the password X-> C1aa[HASH and SALT]
```
 
## Vulnerability-Severity-Assessment 

Using online calculator we can see the CVSS V4.0 score of the potential vulnerability is 7.2. The Consequences are quite variable depending of the type of machine we get into even so this a proof of concept, the ease of use the exploit make this number high.

To obtain this number, I've used this site : https://nvd.nist.gov/vuln-metrics/cvss/v4-calculator, and considered this : CVSS:4.0/AV:P/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:N/SC:H/SI:H/SA:N/E:P/MAV:P/MAC:L/MAT:N/MPR:N/MUI:N/MVC:H/MVI:H/MVA:H/R:U/V:D/RE:M   feel free to see for yourself and discuss it if you disagree.

---
 
## Attack(s) description and implementation
 
### _Simple power analysis_
 
**Requirements** : _physical entry point , microC specialized device like chipwhisperer_
 
#### Principle
 

By using the montage shown above, we can, using the chipwhisperer board, sample a lot of imformation from the µC. Doing so, we can obtain the orange trace over the next picture revealing a little of the internal functionning. By hypothesis we can assume that the verification will occur at the peak between 1600 - 1900 and check for multiple passwords the hypothesis. The blue trace is a password check with the first letter correct (given as hint by our instructor). We can see that the behavior of the trace do not change  but get a less high peak at our hypothethical verification moment, meaning the code react to it, meaning multiple thing like : early-exit or hamming weight correlation or whatever. This peak will act as our **leak**.

<img width="500" height="500" alt="image" src="images/comp200 2.png" />

With the file testSPAfinal.ipynb we can sample multiple trace and get the mean value of them for a knowned character. it allow us to compare multiple mean power between them, And by choosing a reference (a trace made by sending a character we know will never be in the password ) we can obtain a list of the most distant trace in our interesting interval which by extension will show us if a character is different from other, and by hypothesis a character in the password. As we choose our reference as the 100% wrong character. 

By doing that, we can totally and accurately break the password by executing the code multiple code over itself and trying the password obtain in the vault as the code will find byte one by one. 

<img width="947" height="623" alt="image" src="images/password.png" />

## Proposed counter-measure

To counter this exploit we can do lots of things, in this case I will just hash the password and add noise over the power analysis. As now each mean values balance out.

Alternatives we could also not use any early exit code to change the behavior of the code erasing of displacing the leak point.

<img width="600" height="623" alt="image" src="images/countermeasfinal1.png" />
<img width="600" height="623" alt="image" src="images/countermeasfinal2.png" />

## References

- https://github.com/parastuffs/Embedded-Security/wiki/Connecting-to-an-external-target
- https://chipwhisperer.readthedocs.io/en/latest/Capture/20-pin-connector.html
- https://solderingmind.com/atmega328/
  

 
