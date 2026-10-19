
# School_project_arduino_vuln_write_up
 

## SUMMARY
 
1. [Hardware](#hardware)
2. [Asset description](#asset-description)
3. [Attack tree](#attack-tree)
4. [Vulnerability severity assessment](#vulnerability-severity-assessment)
5. [Attack(s) description and implementation](#attacks-description-and-implementation)
6. [Proposed counter-measure(s)](#proposed-counter-measures)
7. [Références](#références)
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

 
![Schéma du montage](Pictures/setup.png)
 
---
 
## Asset description

| Asset | Location | Confidentiality |
|---|---|---|
|key entry location | in one of the entry point | medium as limited by the number of interface |
| Keys | in the µC | high |

The key will let us get into the main architecture. In theory the key hold the most value, nonetheless if the firmware is known and not carefully hash we can use LLMS or other tools to decrypt it making all this attack meaningless.
 
---
 
## Attack tree
 
Objectif de l'attaquant : **This write-up is educational so the goal is just to get into the arduino and obtain one salt and one hash.**
 
```mermaid
graph TD
    A[Objectif : récupérer le secret] --> B[Attaque par canal auxiliaire]

    B --> C1[Analyse de consommation]
    C1 --> C1a[SPA]
    C1a --> C1aa[DPA]
    C1aa --> C1aaa[HASH and SALT]
```
 
## Vulnerability-Severity-Assessment 

Using online calculator we can see the CVSS V4.0 score of the potential vulnerability is 7.2. The Consequences are quite variable depending of the type of machine we get into even so this a proof of concept, the ease of use the exploit make this number high.

To obtain this number, I've used this site : https://nvd.nist.gov/vuln-metrics/cvss/v4-calculator, and considered this : CVSS:4.0/AV:P/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:N/SC:H/SI:H/SA:N/E:P/MAV:P/MAC:L/MAT:N/MPR:N/MUI:N/MVC:H/MVI:H/MVA:H/R:U/V:D/RE:M   feel free to see for yourself and discuss it if you disagree.

---
 
## Attack(s) description and implementation
 
### _Simple power analysis_
 
**Requirements** : _physical entry point , microC specialized device like chipwhisperer_
 
#### Principle
 
_le premier type d'attaque que j'ai réalisé a été une simple Simple power analysis. En se connectant au microcontrôleur avec le montage comme sur la photo au dessus on peut apercevoir sur  la figure ci-dessous, le tracé orange correspondant une série de 200 test d'envoie de mot de passe dans le microcontrolleur._

The simple power analysis goal is to first caractherize the system. By using the montage shown, we can using the chipwhisperer board sample a lot of imformation from the µC. doing so we can obtain the orange trace over the next picture revealing a little of the internal functionning. We see 

![Affichage visuel d'une série d'échantillon (200) effectué par la chipwhisperer ](images/comparaison200.png)

 
#### Summary of the exploit
 
1. _stage 1 : Montage_
2. _stage 2 : analysis_
3. _stage 3 : data processing_
4. _stage 4 : exploit_



 
