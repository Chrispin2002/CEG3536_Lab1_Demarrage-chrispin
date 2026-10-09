# JOURNAL.md — Journal d'équipe, CEG 3536, laboratoire 1 (automne 2026)

Équipe : Anwar Mohammed Halimi et Chrispin Niyomukiza — Section : A01 — Dépôt Git : https://github.com/CEG3536_Lab1_Demarrage chrispin.git

## Jalon J1 (au plus tard le vendredi 25 septembre 2026, validé dans Git)

### Exigences de l'équipe
| Id | Exigence (reformulée par l'équipe) | Critère d'acceptation | Hypothèses |
|---|---|---|---|
| E1 | Initialisation du système | Après réinitialisation (NRST), l'état initial est ARRÊT et seule la DEL rouge (PA9) s'allume en moins de 100 ms. | Niveaux logiques initialisés à la mise sous tension. |
| E2 | Défilement d'état via le bouton User | Chaque appui validé sur le bouton User (PC13) fait alterner les états : ARRÊT → MARCHE_AVANT → ARRÊT → MARCHE_ARRIÈRE → ARRÊT. Une seule DEL est allumée à la fois. | Les transitions passent obligatoirement par l'état ARRÊT. |
| E3 | Anti-rebond et détection de front | L'anti-rebond logiciel filtre les rebonds (fenêtre de 30 ms). 20 appuis consécutifs génèrent exactement 20 événements sans fausse détection ni appui maintenu multiple. | Période de scrutation de 1 ms (`PERIODE_SCRUTATION_MS`). |
| E4 | Arrêt d'urgence prioritaire (E-Stop) | Un appui sur E-Stop (PB2) déclenche l'interruption EXTI2 (priorité 0 du NVIC). L'ISR éteint immédiatement les DEL verte/bleue, allume la DEL rouge et lève `estop_flag` en < 10 ms. | Priorité maximale attribuée à EXTI2 dans le NVIC. |
| E5 | Signalisation de l'état d'urgence | En ARRÊT_URGENCE, la DEL rouge clignote à 2 Hz (250 ms ON, 250 ms OFF ±10%), les DEL verte et bleue restent éteintes et le bouton User est ignoré. | Boucle principale cadencée par `delay_ms`. |
| E6 | Acquittement d'urgence | Un appui validé sur Touch En (PB5) alors que E-Stop est relâché fait repasser le système en ARRÊT (DEL rouge fixe). L'acquittement est refusé si E-Stop est maintenu. | Aucune reprise automatique sans acquittement manuel. |
| E7 | Autorisation Touch En hors urgence | En mode normal, chaque appui validé sur Touch En inverse la variable `touch_enabled` (0/1) et provoque une extinction brève de 100 ms de la DEL active. | `touch_enabled` est stockée en mémoire `.bss`. |
| E8 | Temporisation et calibration | La routine `delay_ms` utilise une boucle de comptage calibrée à l'aide de `DWT_CYCCNT` pour une fréquence d'horloge MSI de 4 MHz. | Horloge système fixe à 4 MHz. |
| E9 | Robustesse du système | En cas d'appuis simultanés ou d'interruptions E-Stop pendant des transitions, le système conserve des états cohérents et n'allume jamais deux DEL en même temps. | EXTI2 est prioritaire sur l'exécution du code principal. |

### Rôles et rotation
| Séance | Réalise | Valide (essais, mesures, relecture) |
|---|---|---|
| Séance 0 (18 sept. 2026) | Chrispin Niyomukiza | Anwar Mohammed Halimi |
| Séance 1 (25 sept. 2026) | Anwar Mohammed Halimi | Chrispin Niyomukiza |
| Séance 2 (2 oct. 2026) | Chrispin Niyomukiza | Anwar Mohammed Halimi |

### Échéancier des laboratoires 1 à 5
| Laboratoire | Séances | Démonstration | Remise | Responsable du suivi |
|---|---|---|---|---|
| 1 | S0 (18 sept), S1 (25 sept), S2 (2 oct) | 2 octobre 2026 | 9 octobre 2026 | Chrispin Niyomukiza |
| 2 | S3, S4 | 23 octobre 2026 | 30 octobre 2026 | Anwar Mohammed Halimi |
| 3 | S5, S6 | 13 novembre 2026 | 20 novembre 2026 | Chrispin Niyomukiza |
| 4 | S7, S8 | 27 novembre 2026 | 4 décembre 2026 | Anwar Mohammed Halimi |
| 5 | S9, S10 | 4 décembre 2026 | 9 décembre 2026 | Chrispin Niyomukiza |

---

## Journal des séances

### Séance 0 — 18 septembre 2026 — réalise : Chrispin Niyomukiza / valide : Anwar Mohammed Halimi
- **Objectifs** : Prise en main de la plateforme Leafy et de la carte Nucleo-L552ZE-Q. Configuration de STM32CubeIDE en assembleur ARM. Création et initialisation du dépôt Git d'équipe.
- **Fait** : Importation du projet de démarrage `CEG3536_Lab1_Demarrage`. Validation de la compilation en assembleur sans HAL. Test de l'allumage de la DEL rouge (PA9) au démarrage.
- **Décisions** : Utilisation de la structure de fichiers fournie (`registres.inc`, `gpio.s`, `buttons.s`, `estop.s`, `fsm.s`).
- **Difficultés et solutions** : Problème initial de connexion du débogueur ST-LINK. Résolu en mettant à jour le micrologiciel ST-LINK dans STM32CubeIDE.
- **Essais et mesures** : Validation T1 (DEL rouge allumée seule après réinitialisation).
- **Validations Git (auteur, message)** :
  - `Chrispin Niyomukiza` : `Initial commit: import Lab1 starter project and setup core files`
  - `Anwar Mohammed Halimi` : `Docs: add team JOURNAL.md template and initial setup`

### Séance 1 — 25 septembre 2026 — réalise : Anwar Mohammed Halimi / valide : Chrispin Niyomukiza
- **Objectifs** : Implémenter les sous-programmes `gpio_init`, `led_set`, `button_raw`, `button_pressed` et la calibration de `delay_ms`. Valider la machine à états E1 à E3 et valider le jalon J1.
- **Fait** : Écriture de la scrutation des boutons et de l'anti-rebond logiciel (fenêtre de 30 échantillons). Lecture du registre IDR pour confirmer les niveaux logiques des boutons.
- **Décisions** : Validation des hypothèses des niveaux logiques actifs : BTN_USER (actif haut), BTN_ESTOP (actif haut), BTN_TOUCH (actif haut). Configuration des résistances de tirage dans `PUPDR`.
- **Difficultés et solutions** : Rebond parasite observé sur le bouton User lors des tests d'appui long. Résolu en ajustant la fenêtre d'anti-rebond `ANTIREBOND_MS` à 30 ms.
- **Essais et mesures** : Validation de T2 (cycle User) et T3 (20 appuis sans erreur, appui maintenu = 1 seule transition). Relevé des entrées IDR pour T4.
- **Validations Git** :
  - `Anwar Mohammed Halimi` : `Feat: implement gpio_init, led_set, and button_raw with debouncing`
  - `Chrispin Niyomukiza` : `Feat: complete fsm.s transitions E1-E3 and validate J1 requirement table`

### Séance 2 — 2 octobre 2026 — réalise : Chrispin Niyomukiza / valide : Anwar Mohammed Halimi
- **Objectifs** : Configuration de l'interruption EXTI2 pour E-Stop (`estop.s`), gestion de l'état `ARRÊT_URGENCE`, clignotement à 2 Hz, acquittement par Touch En et démonstration finale devant l'assistant.
- **Fait** : Programmé `EXTI2_IRQHandler` pour réaliser l'extinction immédiate de LD1 et LD2, l'allumage de LD3 et le positionnement du drapeau `estop_flag`. Finalisé la fonction `fsm_step` et la démonstration.
- **Décisions** : Placement de la mise en sécurité minimale directement dans l'ISR afin de garantir une réactivité sous les 10 ms, conformément à l'exigence E4.
- **Difficultés et solutions** : L'interruption EXTI2 ne se déclenchait pas au premier essai. Correction de l'indexation du registre `EXTI_EXTICR1` et activation du bit 13 dans `NVIC_ISERO`.
- **Essais et mesures** : Réalisation et validation complète des essais T5 à T10. Démonstration effectuée le 2 octobre 2026 avec succès devant l'assistant de laboratoire.
- **Validations Git** :
  - `Chrispin Niyomukiza` : `Feat: configure EXTI2 interrupt, NVIC and E-Stop handler (E4-E6)`
  - `Anwar Mohammed Halimi` : `Fix: fine-tune 2 Hz LED blinking cycle with DWT_CYCCNT and test robustness (T5-T10)`

---

## Tableau des essais (T1 à T10)
| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Réinitialisation | 18/09/2026 | État ARRÊT activé en < 100 ms, DEL rouge (PA9) allumée seule. | Succès | `preuves/T1_Reset_IDR_ODR.png` |
| T2 Cycle User | 25/09/2026 | Séquence R → V → R → B → R respectée à chaque appui sur User. | Succès | `preuves/T2_User_Cycle_ODR.png` |
| T3 Anti-rebond | 25/09/2026 | 20 appuis = 20 transitions exactes ; appui maintenu = 1 seule transition. | Succès | `preuves/T3_Debounce_Watch.png` |
| T4 Niveaux logiques | 25/09/2026 | Relevés IDR : PC13 (User, 1), PB2 (E-Stop, 1), PB5 (Touch, 1). PUPDR validé. | Succès | `preuves/T4_IDR_Levels_Table.png` |
| T5 E-Stop | 02/10/2026 | Entrée immédiate en ISR (< 10 ms), LD1/LD2 éteintes, LD3 allumée, `estop_flag = 1`. | Succès | `preuves/T5_EStop_ISR_Breakpoint.png` |
| T6 Clignotement | 02/10/2026 | Mesure DWT_CYCCNT : 2000000 cycles à 4 MHz = 500 ms de période (2 Hz ±0.5%). | Succès | `preuves/T6_Blink_DWT_Cycles.png` |
| T7 Acquittement | 02/10/2026 | Touch En fait repasser en ARRÊT si E-Stop relâché ; ignoré si E-Stop maintenu. | Succès | `preuves/T7_Ack_Reset_State.png` |
| T8 User ignoré en urgence | 02/10/2026 | Appuis sur User durant `ARRÊT_URGENCE` sans aucun effet sur l'état. | Succès | `preuves/T8_User_Ignored_Emergency.png` |
| T9 Touch En hors urgence | 02/10/2026 | `touch_enabled` alterne (0/1) et extinction de la DEL active pendant 100 ms. | Succès | `preuves/T9_Touch_Toggle_Watch.png` |
| T10 Robustesse | 02/10/2026 | Appuis simultanés et E-Stop pendant clignotement : aucune incohérence constatée. | Succès | `preuves/T10_Robustness_Log.png` |

---

## Routine conservée pour L3-A
- **Routine** : `button_pressed`
- **Interface** :
  - Paramètre : `R0` = identifiant du bouton (0: User, 1: E-Stop, 2: Touch En)
  - Retour : `R0` = 1 si un front d'appui valide est détecté, 0 sinon.
  - Respect des règles AAPCS : sauvegarde/restauration des registres `R4-R11` et de `LR`, pile alignée sur 8 octets.
- **Cas d'essai** :
  1. Entrée avec état stable relâché (`R0 = 0` retourné).
  2. Entrée avec rebonds transitoires < 30 ms (`R0 = 0` retourné).
  3. Entrée avec appui valide maintenu > 30 ms (`R0 = 1` au front montant, puis `R0 = 0` sur les cycles suivants).

---

## Déclaration des sources et de l'usage d'outils d'IA générative
- **Sources** :
  - Manuel de référence STMicroelectronics RM0438 (STM32L552 / STM32L562).
  - Code de démarrage fourni pour le Laboratoire 1 (`CEG3536_Lab1_Demarrage.zip`)[cite: 10].
  - Manuel *Embedded Systems with ARM Cortex-M Microcontrollers in Assembly Language and C* (Yifeng Zhu, 4e édition)[cite: 7].
- **Outils d'IA** :
  - Outil : OpenAI ChatGPT (GPT-4o) / Google Gemini.
  - Usage : Aide à la mise en forme du journal d'équipe Markdown, vérification des règles de syntaxe des directives de l'assembleur GNU `as`, et structuration des tableaux de preuves d'essais. Tout le code assembleur a été testé, vérifié et validé individuellement sur la carte Nucleo-L552ZE-Q[cite: 7, 10].
