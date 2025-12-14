## Project Description

This project demonstrates the secure generation and reliable transmission of a secret key by combining **quantum randomness** with **classical error correction**.

First, quantum random numbers are generated using a quantum circuit. Each qubit is initialized in the ground state and a Hadamard gate is applied, placing the qubit into a superposition of states. When the qubits are measured, the outcomes are fundamentally random, as they arise from the intrinsic principles of quantum mechanics rather than from any deterministic algorithm. These measurement results produce truly random classical bits, which can be used as a secret key.

However, quantum-generated bits are vulnerable to errors during transmission. Any interaction with the environment or transmission medium may cause bit flips, leading to discrepancies between the sender’s and receiver’s keys. To address this issue, redundancy is introduced using a classical error-correcting code.

The secret key is encoded using the **Hamming(7,4) code**, which expands four data bits into a seven-bit codeword by adding three parity bits. This structured redundancy enables the detection and precise localization of a single-bit error. Once the erroneous bit is identified through parity checks, it can be flipped back to recover the original key.

By combining quantum random key generation with Hamming error correction, this project illustrates how quantum randomness can be made reliable for practical communication systems.

