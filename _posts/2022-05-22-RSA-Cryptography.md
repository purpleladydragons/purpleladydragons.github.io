## Goal
We want to communicate securely across an insecure channel. We assume that we're unable to share even a primary secret between two parties without an eavesdropper. So we have to send secure messages from the start.

To do so, we have to use asymmetric key encryption. This is opposed to symmetric key encryption, where both parties know and share the secret key. This means that each person has their own secret key. In practice, asymmetric key exchange is used to securely establish a shared secret key for further communication since symmetric encryption/decryption is faster than asymmetric (TODO explain why later)


We have two major asymmetric systems: ElGamal and RSA. ElGamal is based on Diffie-Hellman key exchange, which is based on discrete logarithms. RSA is based on integer factorization. (TODO explain what discrete logs are)

By "based on", I mean that the security of these systems ultimately depend on how hard it is to solve these underlying problems. 
(And technically, the systems are possibly weaker than these underlying problems since they make public some extra information that makes the problems slightly different)

## RSA

How does RSA work?

### Integer factorization
First off, we've noted that RSA will ultimately use the difficulty of integer factorization as its security measure.
That means that for an eavesdropper to decrypt a private message, it should be (roughly) as hard as factoring some number N.
Clearly, if we choose something trivial for N like 15, then an eavesdropper would very immediately be able to break our code.

So how do we choose a good N? 

Well, it should be big. Integer factorization is still an "unsolved" problem. We don't have any efficient algorithms for doing it, and they probably don't exist, but we don't know for sure (it basically boils down to P=NP). In general, for a number N that is b bits, we don't have any algorithms that run in polynomial time O(b^k). The best algorithms are sub-exponential. The [general number field sieve](https://en.wikipedia.org/wiki/General_number_field_sieve) is an example of the one of the most efficient algorithms.
As an aside, Shor's algorithm provides a polynomial solution for factoring numbers, which means that quantum computers would/could render RSA obsolete as a secure system.

Why can't N be prime itself? (TODO show why it doesn't work in RSA)

The hardest numbers to factor are those that are "semiprime". 
Semiprime means that a number is a product of two primes. 
Why are these the hardest? Well what's the alternative? All numbers factor into primes ultimately. 
Sure, if you multipled it by another prime factor that would be even harder, but the number would be much larger and makes the system slower in general.
When we're compromising for practical speeds for the sender and recipient, we need to be mindful of limiting how large N is. 
For a given rough value of N, the hardest to factor numbers are semiprime. Otherwise it has more factors which are necessarily smaller and possibly repeated.
This makes factoring significantly easier, and many of the most efficient algorithms ultimately relate to the difficulty of finding the smallest prime factor.

#### Some caveats to choosing N
In practical details, there are some extra constraints around p and q because there are several tricks that make factoring N easier under certain conditions. [example](https://crypto.stackexchange.com/questions/13113/how-can-i-find-the-prime-numbers-used-in-rsa)

How does Alice choose p and q though? For modern security, N needs to be at least 1024 bits. This means that p and q should be roughly 512 bits each (TODO why? this is just a basic binary arithmetic quesiton i think)
But these numbers are still massive. It would take a *while* to generate and determine whether they're prime.
NIST publishes prime numbers, so maybe we could use those. But this is still problematic. If you used a public and pre-generated list, 
then suddenly the search space would be very small and it wouldn't be hard for an attacker to find p and q. 
In reality, there does not exist such a list for 512 bit primes. Instead, there is an algorithm to generate large primes.
[This stackexchange answer](https://crypto.stackexchange.com/questions/1970/how-are-primes-generated-for-rsa) does a great job of explaining. 
Essentially, you are likely to find a candidate prime after less than 200 tries starting from a random 512 bit number. 
So that's very quick, to generate one, but to generate all of them, you'd need to do it 2^512 times which is unbelievably huge.
(If you protest that this number includes small primes like 3 and 5 technically, then you can fix the largest bit to 1 and check for the remaining 2^511 options)

### Using N to encrypt and decrypt messages

Okay so we know how to choose a big N now. How do we actually use this information to encrypt and decrypt a message?

First off, realize that a message can be converted to ASCII / bits. Once you encode your message into bits, then that means it has a numerical value in base 10 as well.
This value `m` represents our message, and is what we want to hide from eavesdroppers.

A useful property of modular arithmetic is that exponentiation seems to generate random outputs. This is good for encryption! It means that if we put in two nearly identical messages, we get two totally different results. This means that someone monitoring all the encrypted messages, no matter how many, should not be able to learn anything about the secret messages.
It's generally held in consensus that the outputs are truly pseudorandom and not predictable. But there's no proof yet. 
(TODO include graph of message values and their random cipher outputs on a x-y coordinate graph)

So to encrypt our message m, we will exponentiate it to somer power e, and then mod it by N: c = m^e mod N. 
(TODO why do we use N as the modulus? because uhhh the decryption algorithm relates to the modulus and not to e)

### Choosing e

This is by far the longest section. Now that we've specified N, we have to satisfy some properties when choosing e for everything to work. 
I've tried to present this choice as naturally as possible while also proving the results we need to justify our choice.
One thing that always irks me in mathematical texts is how theorems seem to just pop up out of nowhere. The relevance is rarely apparent at first and often looks like pulling a rabbit out of a hat when you need to. I've tried to limit that experience here, but there are still some foundational number theory results here that useful/needed that I think may appear jarring/convenient unless you were already familiar with them.

We've said already that N should be a very large (1024 bits) semiprime number. But what should e be?
To decrypt the ciphertext, we want to "undo" the encryption of our message m. m^e mod N = c means then that we want to find some decryption key d
such that c^d^e = m mod N, implying de = 1 mod N (in other words, d is the multiplicative inverse of e modulo N)
But does such a d even exist? Yes, if (and only if) e is coprime with N 

Proof:
(TODO first prove thrm 1.11 / Bezout's identity)

assume gcd(e, N) = 1
With Bezout's identity we get ed + kN = 1. Thus ed - 1 = -kN is divisible by N,
therefore ed = 1 mod N. 

Note that d is unique; otherwise would mean ed_1 = ed_2 = 1 mod N, which means that e(d_1 - d_2) = 0 mod N, since e is coprime to N, then we can divide by e and we see d_1 - d_2 = 0, so d_1 = d_2 and we end up with a unique d anyway)

Great! But there's a further issue we need to address: we need to make sure the decryption is unique. 
This means that given a cipher c, c^d = m should be unique. Otherwise the recipient wouldn't be able to decipher the text into a single message.
(Note of course that just because d is unique doesn't mean c^d mod N has to be. For the trivial example, if d = N-1 (with prime N), then c^d = 1 mod N for all c)

So how do we guarantee that c^d is unique? We need to choose an e such that c^d has this property. And how do we do that?

Let's first introduce Euler's formula.
Euler's formula is a function phi defined as phi(n) = the number of positive integers less than n that are coprime to n.
We're going to focus only on phi(n) for prime n. In which case, phi(n) = n-1 trivially because of the definition of n being prime: there are no numbers less than n that divide n. 

But we're focused on N=pq, so N is not prime.

Luckily, phi(N) is multiplicative in this case meaning that phi(pq) = phi(p)phi(q).
To show so we leverage the Chinese remainder theorem:

CRT Proof:
We'll show a weak version of the CRT here instead of the fully generalized version. If we have two numbers m and n that are coprime (gcd(m,n) = 1),
then the system x = a % m and x = b % n has a unique solution.

Proof of existence:
From the first congruence: x = my + a
We can plug this into the second congruence to get my + a = b % n
my = b - a % n
Because gcd(m,n) = 1, we know that m has an inverse modulo n. Let's call it m'
Then m'my = m'(b-a) % n 
We can cancel m'm because we're showing congruence modulo n: y = m'(b-a) % n (TODO can we? idk why)
Rewrite this as multiplication with remainder: y = zn + m'(b-a)
Plug this back into the first equation x = my + a: x = m(zn + m'(b-a)) + a
We can't reduce this further (because we're no longer working in modulo, so m' is not generally the inverse of m)
But we've shown that the solution for x in this system has this form

Proof of uniqueness:
Assume x = c and x = c' both solve the system
Then c is congruent to c' mod m, ie c = c' % m 
So c = my + c', then c - c' = my
So (c - c') is divisible by m
We can repeat the same line of reasoning for n instead of m, so that (c - c') is divisible by n
Because gcd(m,n) = 1 then this means that mn also divides (c - c')
Which means c = c' % mn
In other words, x = c and x = c' implies c is unique modulo mn

This is important because it allows us to construct a bijection which shows that phi is multiplicative.
Specifically, say we have distinct primes p and q, and say we have the sets A,B,C where A is the set of integers coprime to p, B is the set of integers coprime to q, and C is the set of integers coprime to pq. So |A| = phi(p), |B| = phi(q), and |C| = phi(pq)
If we can demonstrate a bijection between AxB and C, then that means that phi is multiplicative and we know that phi(pq) = phi(p)phi(q)

The set AxB is every pair (a,b) such that gcd(a,p) = 1 and gcd(b,q) = 1 where a < p and b < q.
We take every pair (a,b) and run it through f(a,b): y = a % p and y = b % n.
From the CRT, we know that each pair maps to exactly one solution. So we have a bijection from AxB to some set of integers D.

But we can show that D = C: if d is a solution to f(a,b) then that means d is coprime to both p and q. Since gcd(p,q) = 1 then d is also coprime to pq. 
That's our definition of the set C! So that means D = C, so we now have a bijection from AxB to C which allows us to treat phi as a multiplicative function

Since p and q are both prime, we know phi(pq) = (p-1)(q-1).

Now we can use thrm 3.1 (TODO, proof below but don't call it such lol) to show a^(p-1)(q-1) = 1 mod pq when a is coprime with pq.
Proof:
a^(p-1)(q-1) = (a^(p-1))^(q-1)
By FLT, we get a^(p-1)^(q-1) = 1^(q-1) mod p
= 1 mod p b/c 1 to anything = 1
You can repeat same process with p and q flipped
So then a^(p-1)(q-1) = 1 mod p and mod q implies a^(p-1)(q-1) - 1 is divisible by both p and q, which means it's divisible by pq as well
therefore a^(p-1)(q-1) = 1 mod pq as well

This was arguably a roundabout way to avoid some group theory. Instead of focusing specifically on phi for prime numbers, using CRT to demonstrate multiplicativity, and using FLT to establish modulo behavior, we could've shown, using Lagrange's theorem, that phi(n) is the order of multiplicative group of modulo n. (Good luck with that lol)

Start with c^d^e = m = m^e^d mod N
From theorem above, we know m^phi(N) = 1 mod N (TODO remove the phi reference etc if you need to)
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
But sometimes a larger e is chosen for more security (TODO remind me of the other e choice = 65167 or whatever)

### Finding the decryption key d!
TODO use Euclidean algorithm with knowledge of p and q to find d so that we can therefore decrypt m^e mod N
TODO also show why Eve can't do shit without p and q... (idk if the book actually covers this? i think it's literally an open problem)

## Practical matters
TODO start coding shit up




