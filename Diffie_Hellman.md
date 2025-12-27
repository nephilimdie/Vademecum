# Diffie-Hellman Key Exchange

## What it is
The Diffie-Hellman key exchange (DH) is a cryptographic method that allows two parties to establish a **shared secret key** over an insecure communication channel.

- Year: 1976  
- Inventors: Whitfield Diffie and Martin Hellman  
- Historical importance: first practical public key cryptography mechanism  

Diffie-Hellman does **not** encrypt data. Its only purpose is to allow two parties to agree on the same secret value, called a *key*, without ever sending that key directly.

This key is later used by symmetric encryption algorithms to encrypt and decrypt data.

---

## The concept of a key
A **cryptographic key** is similar to the key of a lock:

- The lock is the encryption algorithm (AES, ChaCha20, etc.)
- The key determines how the data is locked and unlocked

If two parties have the same key:
- One can encrypt data
- The other can decrypt it

Diffie-Hellman exists to solve one specific problem:
How can two parties obtain the same secret key if everyone can see their communication?

---

## How it works
The protocol relies on the difficulty of the discrete logarithm problem: certain mathematical operations are easy to perform, but extremely hard to reverse.

### Steps
1. Public parameters  
   - A large prime number `p`  
   - A generator `g`  
   These values are public.

2. Private secrets  
   - Alice chooses a secret value `a`  
   - Bob chooses a secret value `b`  

3. Public exchange  
   - Alice computes `A = g^a mod p` and sends it to Bob  
   - Bob computes `B = g^b mod p` and sends it to Alice  

4. Shared key computation  
   - Alice computes `K = B^a mod p`  
   - Bob computes `K = A^b mod p`  

Both obtain the same value:
`K = g^(ab) mod p`

5. Result  
The value `K` is the **shared secret key**.  
It is never transmitted over the network.

---

## Color analogy
The protocol can be understood using a color-mixing analogy.

1. A public color is chosen  
   - Yellow is public and visible to everyone.

2. Each party chooses a secret color  
   - Alice chooses Blue  
   - Bob chooses Red  

3. First mixing step  
   - Alice mixes Yellow + Blue and sends the result to Bob  
   - Bob mixes Yellow + Red and sends the result to Alice  

4. Second mixing step  
   - Alice adds Blue to Bobâ€™s mixture  
   - Bob adds Red to Aliceâ€™s mixture  

5. Final result  
Both obtain the same final color.

The final color represents the shared secret key.
Mixing colors is easy, but separating them back into the original components is extremely difficult.

---

## Applications
- TLS/SSL (HTTPS)
- SSH secure connections
- VPN protocols such as IPSec and WireGuard
- Secure messaging systems

Diffie-Hellman is used only during the initial handshake phase.  
Once the key is established, fast symmetric encryption algorithms are used.

---

## Example with small numbers
Parameters:
- `p = 23`
- `g = 5`

Process:
- Alice chooses `a = 6` and computes `A = 5^6 mod 23 = 8`
- Bob chooses `b = 15` and computes `B = 5^15 mod 23 = 19`

Shared secret:
- Alice computes `K = 19^6 mod 23 = 2`
- Bob computes `K = 8^15 mod 23 = 2`

Final shared key: `2`

---

## Example in Python
```python
p = 23
g = 5

a = 6
b = 15

A = pow(g, a, p)
B = pow(g, b, p)

K_alice = pow(B, a, p)
K_bob = pow(A, b, p)

print(K_alice, K_bob)
```

---

## Advantages and limitations
Advantages:
- No pre-shared secrets required
- Secure when large parameters are used
- Foundation of modern cryptography

Limitations:
- Vulnerable to man-in-the-middle attacks without authentication
- Does not encrypt data directly
- Computationally heavier than symmetric cryptography

---

## Evolutions
- DHE (Diffie-Hellman Ephemeral): temporary keys and forward secrecy
- ECDH (Elliptic Curve Diffie-Hellman): smaller keys with equivalent security
- Post-quantum key exchange: algorithms designed to resist quantum attacks