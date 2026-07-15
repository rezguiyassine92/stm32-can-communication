<div align="center">

# CAN Bus Communication — Two STM32F407 Nodes

### Real-time distributed LED synchronization over CAN 2.0, with MCP2551 transceivers

![STM32](https://img.shields.io/badge/-STM32F407-03234B?style=for-the-badge&logo=stmicroelectronics)
![CAN](https://img.shields.io/badge/-CAN_2.0A-orange?style=for-the-badge)
![MCP2551](https://img.shields.io/badge/-MCP2551-blue?style=for-the-badge)
![C](https://img.shields.io/badge/-C-00599C?style=for-the-badge&logo=c)

`Embedded Communication` · `Multi-Master Bus` · `Interrupt-Driven RX` · `Physical Layer`

</div>

---

## 📋 Overview

Two STM32F407 boards communicating over a real CAN 2.0 bus, using MCP2551 transceivers to drive the physical differential line. Each board runs the same firmware and acts as a peer — there is no master. Pressing a button on one node updates its local state and broadcasts the change; the other node receives the frame through a hardware interrupt and mirrors it in real time.

The system was validated end-to-end: bit-timing configuration, hardware ID filtering, interrupt-driven reception, and bus arbitration when both nodes transmit at once.

## ⚙️ Key Contributions

- 🔗 Configuration and implementation of the CAN protocol on both nodes
- ⚙️ MCP2551 transceivers used for physical bus interfacing
- 📡 Bidirectional real-time data exchange between two independent STM32F407 boards
- 🛠️ System validated under real communication conditions, captured on a PicoScope

## 🔧 Hardware

| Composant | Rôle |
|---|---|
| STM32F407VG ×2 | Nœuds CAN (ARM Cortex-M4) |
| MCP2551 ×2 | Transceiver CAN (ISO 11898, jusqu'à 1 Mb/s) |
| Résistances 120 Ω ×2 | Terminaison de bus (CAN_H / CAN_L) |

### Câblage

| STM32 | → | MCP2551 |
|---|---|---|
| PB9 (CAN1_TX) | → | TXD |
| PB8 (CAN1_RX) | → | RXD |
| 5V | → | VDD |
| GND | → | VSS |

Puis relier les deux transceivers : `CANH ↔ CANH`, `CANL ↔ CANL`, avec la résistance 120 Ω aux deux extrémités du bus.

<div align="center">
  <img src="images/pinout-can-tx.png" width="500" alt="Brochage CAN1_TX / CAN1_RX sur STM32F407VGTx">
  <br><em>Configuration des broches CAN1_TX (PB9) / CAN1_RX (PB8) sur le STM32F407VGTx</em>
</div>

## ⚡ Configuration CAN

| Paramètre | Valeur |
|---|---|
| Périphérique | CAN1 (PB8 RX / PB9 TX, AF9) |
| Mode | Normal |
| Auto-retransmission | Désactivée |
| Horloge CAN (APB1) | 42 MHz |
| Prescaler | 16 |
| Débit nominal | ≈ 375 kbit/s |
| Format de trame | CAN 2.0A — ID standard 11 bits |
| Réception | Interruption (CAN1_RX0, FIFO0) |

La réception passe par un filtre matériel (masque d'ID) qui route les trames vers la FIFO0 : le CPU n'est interrompu que pour le trafic pertinent, sans scrutation active du bus.

## 🔬 Validation — capture oscilloscope

Trame CAN réelle capturée au PicoScope pendant l'échange entre les deux nœuds.

<div align="center">
  <img src="images/oscilloscope-can-frame.png" width="700" alt="Capture oscilloscope d'une trame CAN réelle">
  <br><em>Signal numérique du bus CAN capturé en conditions réelles de communication</em>
</div>

## 🧠 Arbitrage du bus

Le CAN est multi-maître grâce à un arbitrage binaire non destructif : si deux nœuds émettent en même temps, chacun envoie son identifiant bit par bit tout en lisant le bus. Un bit dominant (0) l'emporte toujours sur un bit récessif (1) : dès qu'un nœud envoie un bit récessif mais lit un bit dominant, il sait qu'il a perdu l'arbitrage, s'arrête et devient récepteur — sans corrompre la trame gagnante. Pas d'arbitre central, pas de collision perdue.

## 🚀 Build & Flash

1. Ouvrir le projet dans STM32CubeIDE
2. Compiler en configuration Debug
3. Flasher chaque carte via ST-LINK
4. Donner à chaque nœud une identité distincte (inverser `OwnID` / `RemoteID` sur la seconde carte)
5. Relier les deux transceivers avec la terminaison 120 Ω aux deux extrémités
6. Appuyer sur le bouton — les deux LEDs doivent suivre en miroir

## 🛠 Tech Stack

`STM32F407` · `CAN 2.0A` · `MCP2551` · `STM32 HAL` · `Interrupt-driven RX` · `Hardware ID filtering` · `STM32CubeIDE`

## 📄 License

MIT
