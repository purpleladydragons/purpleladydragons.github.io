## Goal
We want to communicate securely across an insecure channel. We assume that we're unable to share even a primary secret between two parties without an eavesdropper. So we have to send secure messages from the start.

To do so, we have to use asymmetric key encryption. This is opposed to symmetric key encryption, where both parties know and share the secret key. This means that each person has their own secret key. In practice, asymmetric key exchange is used to securely establish a shared secret key for further communication since symmetric encryption/decryption is faster than asymmetric (TODO explain why later)


We have two major asymmetric systems: ElGamal and RSA. ElGamal is based on Diffie-Hellman key exchange, which is based on discrete logarithms. RSA is based on integer factorization. (TODO explain what discrete logs are)

By "based on", I mean that the security of these systems ultimately depend on how hard it is to solve these underlying problems. 
(And technically, the systems are possibly weaker than these underlying problems since they make public some extra information that makes the problems slightly different)

## RSA

How does RSA work?

First off, we've noted that it will ultimately use the difficulty of integer factorization as its security measure.
That means that for an eavesdropper to decrypt a private message, it should be (roughly) as hard as factoring some number N.
Clearly, if we choose something trivial for N like 15, then an eavesdropper would very immediately be able to break our code.

So how do we choose a good N? 

Well, it should be big. Factoring a number grows (TODO) exponentionally in the number of bits of the number (TODO exponential over bits doesn't that just mean exp(log) which is linear?)

The hardest numbers to factor are those that are "semiprime". (TODO why can't N be prime itself?)
Semiprime means that a number is a product of two primes. (TODO why isn't three primes even harder?)
Why are these the hardest? Well what's the alternative? All numbers factor into primes ultimately. 
A number close to N might have many more prime factors. This necessarily means the factors are smaller, and possibly repeated.
This makes factoring significantly easier (it's easier to find smaller factors by brute force, and every time you do, you reduce the search space of further factors)

Okay so we know how to choose a big N now. How do we actually use this information to encrypt and decrypt a message?

First off, realize that a message can be converted to ASCII / bits. Once you encode your message into bits, then that means it has a numerical value in base 10 as well.
This value `m` represents our message, and is what we want to hide from eavesdroppers.

A useful property of modular arithmetic is that exponentiation seems to generate random outputs. This is good for encryption! It means that if we put in two nearly identical messages, we get two totally different results. This means that someone monitoring all the encrypted messages, no matter how many, should not be able to learn anything about the secret messages.
It's generally held in consensus that the outputs are truly pseudorandom and not predictable. But there's no proof yet. 
(TODO include graph of message values and their random cipher outputs on a x-y coordinate graph)

So to encrypt our message m, we will exponentiate it to somer power e, and then mod it by N: c = m^e mod N. 
(TODO why do we use N as the modulus? because uhhh the decryption algorithm relates to the modulus and not to e)

We've said already that N should be a very large (1024 bits) semiprime number. But what should e be?
To decrypt the ciphertext, we want to "undo" the encryption of our message m. m^e mod N = c means then that we want to find some decryption key d
such that c^d^e = m mod N, implying de = 1 mod N (in other words, d is the multiplicative inverse of e modulo N)
But does such a d even exist? Yes, iff e is coprime with N (TODO proof).
Note that d is unique; otherwise would mean ed_1 = ed_2 = 1 mod N, which means that e(d_1 - d_2) = 0 mod N, since e is coprime to N, then we can divide by e and we see d_1 - d_2 = 0, so d_1 = d_2 and we end up with a unique d anyway)
Great! But there's a further issue we need to address: we need to make sure the decryption is unique. 
This means that given a cipher c, c^d = m should be unique. Otherwise the recipient wouldn't be able to decipher the text into a single message.
(Note of course that just because d is unique doesn't mean c^d mod N has to be. For the trivial example, if d = N-1, then c^d = 1 mod N for all c)

So how do we guarantee that c^d is unique? We need to choose an e such that c^d has this property. And how do we do that?

TODO develop Euler's formula and Euler's theorem
formula describes function phi(n) = how many numbers up to n are coprime with n?
in the special case where n is prime, we can use FLT to show that phi(n) = n-1
but otherwise we have two options:
1) use the citations 4 and 5 to show that phi is multiplicative so that we can show phi(N) = (p-1)(q-1)
2) develop Lagrange's theorem and group theory to show that phi(n) is the order of the multiplicative group of integers mod n

Start with c^d^e = m = m^e^d mod N
From Euler's theorem, we know m^phi(N) = 1 mod N
Take them to the kth power, m^kphi = 1 mod N (because 1^k = 1)
Multiply both sides by m, m^(kphi + 1) = m mod N
m^(kphi + 1) = m^ed mod N
kphi + 1 = ed
This is important! This basically is another way of saying ed = 1 mod phi(N)
Since phi is multiplicative, phi(N) = phi(p)phi(q) = (p-1)(q-1)
So ed = 1 mod (p-1)(q-1)
And from the same prop as above, we know then that gcd(e, (p-1)(q-1)) = 1

So going back again, assume m = u is a solution to m^e = c mod N.
Because k(p-1)(q-1) + 1 = de, and since u = u^1 obviously, then we can get u = u^(de - kphi)
= u^ed * u^((p-1)(q-1)^-k) mod pq
= u^ed * 1^-k mod pq ; bc of Euler's formula 
= c^d mod pq ; bc u^e = c and 1^-k = 1
Therefore c^d is unique solution when gcd(e, (p-1)(q-1)) = 1

So, p-1 and q-1 are both even numbers, and therefore e=3 satisfies the requirement. This actually gets used in practice!
But sometimes a larger e is chosen for more security.

-----

Alice wants to create her public key.
She does so by choosing two secret values p and q. p and q should be very large primes. In practical details, there are some extra constraints around p and q because there are several tricks that make factoring N easier under certain conditions. [example](https://crypto.stackexchange.com/questions/13113/how-can-i-find-the-prime-numbers-used-in-rsa)

How does Alice choose p and q though? For modern security, N needs to be at least 1024 bits. This means that p and q should be roughly 512 bits each (TODO why? this is just a basic binary arithmetic quesiton i think)
But these numbers are still massive. It would take a *while* to generate and determine whether they're prime.
NIST publishes prime numbers, so maybe we could use those. But this is still problematic. If you used a public and pre-generated list, 
then suddenly the search space would be very small and it wouldn't be hard for an attacker to find p and q. 
In reality, there does not exist such a list for 512 bit primes. Instead, there is an algorithm to generate large primes.
[This stackexchange answer](https://crypto.stackexchange.com/questions/1970/how-are-primes-generated-for-rsa) does a great job of explaining. 
Essentially, you are likely to find a candidate prime after less than 200 tries starting from a random 512 bit number. 
So that's very quick, to generate one, but to generate all of them, you'd need to do it 2^512 times which is unbelievably huge.
(If you protest that this number includes small primes like 3 and 5 technically, then you can fix the largest bit to 1 and check for the remaining 2^511 options)






