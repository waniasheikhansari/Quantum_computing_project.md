## Project Description

This project demonstrates the generation of a secret key by combining **quantum randomness** with **classical error correction**.

First, quantum random numbers are generated using a quantum circuit. Each qubit is initialized in the ground state and a Hadamard gate is applied, placing the qubit into a superposition of states. When the qubits are measured, the outcomes are random, as they arise from the principles of quantum mechanics rather than from any pre-define algorithm. These measurement results produce truly random classical bits, which can be used as a secret key.

However, quantum-generated bits are sensitve to errors during transmission. Any interaction with the environment or transmission medium may cause bit to flip and this would crroupte the bit.

The secret key is encoded using the **Hamming(7,4) code**, which expands four data bits into a seven-bit key by adding three parity bits. This enables the detection and precise localization of a single-bit error. Once the crroupted bit is identified through parity checks, it can be flipped back to recover the original key.

By combining quantum random key generation with Hamming error correction, this project illustrates how quantum randomness can be made reliable for practical communication systems.

