## STEP 1: Quantum Random Key Generation
Each qubit is initialized in the state  
|0>
A Hadamard gate is applied:
H|0> = (|0> + |1>) / sqrt(2) which would convert initial state as the superposition of 2 states.
Before measurement result is completely probabilistic but Upon measurement the outcome will become **uniformly random classical bit**,  

### Code
```python
qc = QuantumCircuit(4, 4)

for i in range(4):
    qc.h(i)   #applicatoin of hamming
for i in range(4):
    qc.measure(i, i)  #measurement of the qubit

sim = AerSimulator()
result = sim.run(qc, shots=1).result()
bits= list(result.get_counts().keys())[0][::-1]

mes = []
for b in bits:
    mes.append(int(b))

print("\nQuantum-random key:", mes)
```
# Output

```
Quantum random key : [1, 1, 1, 0]
```
## Step 2: HAMMING(7,4) ENCODING
### Bit order: [p1 p2 d1 p3 d2 d3 d4]

To protect the 4-bit key against noise, it can be encode using the **Hamming(7,4)** error-correcting code.  
This code adds **three parity bits** to the original data, allowing **single-bit error correction**.

The parity bits are computed using XOR operations:

- `p1 = d1 ⊕ d2 ⊕ d4`
- `p2 = d1 ⊕ d3 ⊕ d4`
- `p3 = d2 ⊕ d3 ⊕ d4`
---

### Code
```python
d1, d2, d3, d4 = message
p1 = d1 ^ d2 ^ d4
p2 = d1 ^ d3 ^ d4
p3 = d2 ^ d3 ^ d4

key = [p1, p2, d1, p3, d2, d3, d4]
print("Encoded key:", key)
```

# Output
```
Encoded key: [1, 0, 0, 0, 0, 1, 1]
```
## Step 3: Hamming(7,4) Encoding
The quantum-generated classical key is transmitted through a noisy classical communication channel and quantum system is sensitive to interaction.
Due to the pressence of noise any bit may flip and flipping of the bit will add error in original key.

### Code
```python
rx = key[:]
flip = None

if random.random() < 0.7:
    flip = random.randrange(7)
    rx[flip] ^= 1

print("Received key:", rx)
print("flipped index:", flip if flip is not None else "None")
```
# Output 
``` Received key: [1, 1, 0, 0, 0, 1, 1]
flipped index: 1
Error detected at index: 1
```

## Step 4: Error Detection and Correction Using Syndrome Decoding

Now recevier has obtained the key but noise has corrupted it already now hamming syndrome decoding can be used to locate the error by checking parity rules.

The syndrome bits are computed as follows:

- `s1 = r1 ⊕ r3 ⊕ r5 ⊕ r7`
- `s2 = r2 ⊕ r3 ⊕ r6 ⊕ r7`
- `s3 = r4 ⊕ r5 ⊕ r6 ⊕ r7`

The error position is then interpreted as:
---

### Code
```python
s1 = rx[0] ^ rx[2] ^ rx[4] ^ rx[6]
s2 = rx[1] ^ rx[2] ^ rx[5] ^ rx[6]
s3 = rx[3] ^ rx[4] ^ rx[5] ^ rx[6]

error= s1 + 2*s2 + 4*s3   # 1..7 or 0

if error == 0:
    print("No error detected")
    index = None
else:
    index = error - 1
    print("Error detected at index:", index)

# Correct the error if detected
corrected = rx[:]
if index is not None:
    corrected[index] ^= 1

print("Corrected key:", corrected)
```
# Output 
```
Corrected key: [1, 0, 0, 0, 0, 1, 1]

```
## Step 6: Final Decoding and Message Recovery

After error correction, the receiver attempts to recover the original 4-bit message from the corrected Hamming(7,4) key.

---

### Code
```python
decoded = [corrected[2], corrected[4], corrected[5], corrected[6]]

print("Recovered message:", decoded)
print("Recovered correctly?:", decoded == message)
```

# Output
```
Recovered message: [0, 0, 1, 1]
Recovered correctly?: True
```
# Step 7: Visuals
To enchance readilbity quantum circuit diagram has been added 

# Code
```
def bits_str(x): 
    return "".join(map(str, x))

def draw_flow(message, key, rx, corrected, decoded, flip, index):
    fig, ax = plt.subplots(figsize=(14, 3.8))
    ax.set_title(f"Generated message = {bits_str(message)}   |   Decoded = {bits_str(decoded)}", fontsize=16)

    rows = [
        ("Original (4)",  message,   [0,1,2,3]),
        ("Encoded (7)",   key,       [0,1,2,3,4,5,6]),
        ("Received (7)",  rx,        [0,1,2,3,4,5,6]),
        ("Corrected (7)", corrected, [0,1,2,3,4,5,6]),
        ("Decoded (4)",   decoded,   [0,1,2,3]),
    ]

    y = [4,3,2,1,0]

    for (name, bits, xs), yy in zip(rows, y):
        ax.text(-1.7, yy, f"{name}: {bits_str(bits)}", fontsize=13, va="center")
        for x, b in zip(xs, bits):
            ax.text(x, yy, str(b), ha="center", va="center",
                    bbox=dict(boxstyle="circle,pad=0.35", fc="lightgray", ec="black"))

    # mark flipped in received
    if flip is not None:
        ax.scatter([flip], [2], s=900, facecolors="none", edgecolors="red", linewidths=3)
        ax.text(flip, 1.55, "flipped", color="red", ha="center", fontsize=11)

    # mark detected in corrected
    if index is not None:
        ax.scatter([index], [1], s=900, facecolors="none", edgecolors="green", linewidths=3)
        ax.text(index, 0.55, "detected", color="green", ha="center", fontsize=11)

    ax.set_xlim(-1.8, 6.5)
    ax.set_ylim(-0.6, 4.6)
    ax.set_xticks(range(7))
    ax.set_yticks([])
    ax.grid(axis="x", color="0.9")
    plt.tight_layout()
    plt.show()

draw_flow(message, key, rx, corrected, decoded, flip, index)

circuit_drawer(qc, output="mpl")
fig.savefig("quantum.png", dpi=200, bbox_inches="tight")
plt.show()
```


<img width="2850" height="887" alt="key_flow_visual" src="https://github.com/user-attachments/assets/c88fac1f-133f-40c9-b92b-33215fa4b3af" />




















