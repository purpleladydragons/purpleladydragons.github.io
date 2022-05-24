## Goal
We want to communicate securely across an insecure channel. We assume that we're unable to share even a primary secret between two parties without an eavesdropper. So we have to send secure messages from the start.

To do so, we have to use asymmetric key encryption. This is opposed to symmetric key encryption, where both parties know and share the secret key. This means that each person has their own secret key. In practice, asymmetric key exchange is used to securely establish a shared secret key for further communication since symmetric encryption/decryption is faster than asymmetric (TODO explain why later)

We have two major asymmetric systems: ElGamal and RSA. 
ElGamal is based on Diffie-Hellman key exchange, which is based on discrete logarithms. 
RSA is based on integer factorization. 

By "based on", I mean that the security of these systems ultimately depend on how hard it is to solve these underlying problems. 
(And technically, the systems are possibly weaker than these underlying problems since they make public some extra information that makes the problems slightly different)

## RSA

How does RSA work?

### Integer factorization
First off, we've noted that RSA will ultimately use the difficulty of integer factorization as its security measure.
That means that for an eavesdropper to decrypt a private message, it should be (roughly) as hard as factoring some number N.
Clearly, if we choose something trivial for N like 15, then an eavesdropper would very immediately be able to break our code.

So how do we choose a good N? 

Well, it should be big. Integer factorization is still an "unsolved" problem. 
We don't have any efficient algorithms for doing it, and they probably don't exist, but we don't know for sure (it basically boils down to P=NP). 

In general, for a number N that is b bits, we don't have any algorithms that run in polynomial time $$O(b^k)$$. 
The best known algorithms are sub-exponential. 
The [general number field sieve](https://en.wikipedia.org/wiki/General_number_field_sieve) is an example of the one of the most efficient algorithms.
As an aside, Shor's algorithm provides a polynomial solution for factoring numbers, which means that quantum computers would/could render RSA obsolete as a secure system.

The hardest numbers to factor are those that are "semiprime". 
Semiprime means that a number is a product of two primes. 
Why are these the hardest? Well what's the alternative? All numbers factor into primes ultimately. 
Sure, if you multipled it by another prime factor that would be even harder, but the number would be much larger and makes the system slower in general.
When we're compromising for practical speeds for the sender and recipient, we need to be mindful of limiting how large N is. 
For a given rough value of N, the hardest to factor numbers are semiprime. Otherwise it has more factors which are necessarily smaller and possibly repeated.
This makes factoring significantly easier, and many of the most efficient algorithms ultimately relate to the difficulty of finding the smallest prime factor.

So ultimately we have to choose two distinct (and large) primes p and q, then N=pq.

> **Why can't N itself be prime?**
>
> N cannot be prime for two reasons: we have to make N public and if N were prime, then anyone who knows N can easily decipher the message.
> 
> The reason that it's easy to decipher the message once you know N (if it's prime) is covered farther down in the section "Finding the decryption key d"

> **Some caveats to choosing N**
>
> In practical details, there are some extra constraints around p and q because there are several tricks that make factoring N easier under certain conditions. [Stack exchange provides an example.](https://crypto.stackexchange.com/questions/13113/how-can-i-find-the-prime-numbers-used-in-rsa)
>
> How does one choose p and q though? For modern security, N needs to be at least 1024 bits. This means that p and q should be roughly 512 bits each. 
> 
> But these numbers are still massive. It would take a *while* to generate and determine whether they're prime.
> 
> NIST publishes prime numbers, so maybe we could use those. But this is still problematic. If you used a public and pre-generated list, 
then suddenly the search space would be very small and it wouldn't be hard for an attacker to find p and q. 
> 
> In reality, there does not exist such a list for 512 bit primes. Instead, there is an algorithm to generate large primes.
[This stackexchange answer](https://crypto.stackexchange.com/questions/1970/how-are-primes-generated-for-rsa) does a great job of explaining.
> 
> Essentially, you are likely to find a candidate prime after less than 200 tries starting from a random 512 bit number. 
So that's very quick, to generate one, but to generate all of them, you'd need to do it 2^512 times which is unbelievably huge.
(If you protest that this number includes small primes like 3 and 5 technically, then you can fix the largest bit to 1 and check for the remaining 2^511 options)

### Using N to encrypt and decrypt messages

Okay so we know how to choose a big N now. How do we actually use this information to encrypt and decrypt a message?

First off, realize that a message can be converted to ASCII / bits. Once you encode your message into bits, then that means it has a numerical value in base 10 as well.
This value `m` represents our message, and is what we want to hide from eavesdroppers.

A useful property of modular arithmetic is that exponentiation seems to generate random outputs. This is good for encryption! It means that if we put in two nearly identical messages, we get two totally different results. This means that someone monitoring all the encrypted messages, no matter how many, should not be able to learn anything about the secret messages.
It's generally held in consensus that the outputs are truly pseudorandom and not predictable. But there's no proof yet. 

<p align="center">
<img src="https://raw.githubusercontent.com/purpleladydragons/purpleladydragons.github.io/master/images/rsa-cryptography/rsa-exp-mod-random.png" width="340"/>
</p>

<p align="center"><i>Applying y=x^e mod N for several thousand x with e=3 and N=30402457</i></p>


So to encrypt our message m, we will exponentiate it to some power e, and then mod it by N: $$c \ \equiv \ m^e \pmod{N}$$. 
(TODO why do we use N as the modulus? because uhhh the decryption algorithm relates to the modulus and not to e)

### Choosing e

This is by far the longest section. Now that we've specified N, we have to satisfy some properties when choosing e for everything to work. 
I've tried to present this choice as naturally as possible while also proving the results we need to justify our choice.
One thing that always irks me in mathematical texts is how theorems seem to just pop up out of nowhere. 
The relevance is rarely apparent at first and often looks like pulling a rabbit out of a hat when you need to.
I've tried to limit that experience here, 
but there are still some foundational number theory results here that are useful/needed 
that I think may appear jarring/convenient unless you were already familiar with them.

We've said already that N should be a very large (1024 bits) semiprime number. But what should e be?

To decrypt the ciphertext, we want to "undo" the encryption of our message m. $$m^e \pmod{N} = c$$ means then that we want to find some decryption key d
such that $$c^{d^e} \ \equiv \ m \pmod{N}$$, implying $$de \ \equiv \ 1 \pmod{N}$$ (in other words, d is the multiplicative inverse of e modulo N)

But does such a d even exist? Yes, if (and only if) e is coprime with N 

----
### Proving e has an inverse modulo d when e is coprime with N

First we have to prove Bezout's identity, a foundational result in number theory.

Bezout's identity states that for two integers a and b with gcd(a,b) = d, 
then there exist integers x and y such that ax + by = d.
We'll actually prove a slightly weaker version of the identity by assuming that a and b are positive.

Given positive integers a and b, we construct the set S {ax + by | x,y s.t ax + by > 0}.

S is non-empty since y=0, x=1 satisfies the set criterion for all positive a. S also has a minimum d because it is a set of positive integers.
We want to show that d is also the gcd of a and b.

To do so, we have to show that d divides a and b, and that for any other divisor c of a and b, c <= d.

#### Show d divides a and b

We can write the division of a by d as a = nd + r.

Rewritten: r = a - nd.

Remember that d is an element of S, so we can rewrite d as ax + by for some x and y.

Thus r = a - n(ax + by)

= a - nax - nby

= a(1 - nx) + (-ynb)

So we've rewritten r in the form of ax + by. 
We also know r is non-negative because since d is minimum of S, then d <= a so we can always choose a non-negative r satisfying nd + r = a.

But we also know r < d, because otherwise we rewrite a = nd + r as a = (n+1)d + (r-d).
Since d is the minimum of S and is positive, then r has to be 0.

Therefore d divides a. We can repeat the same arugment for b.

To show that d is the greatest common divisor, we have to show that for any other c that divides a and b, c <= d.

Assume c divides a and b. Then a = cx and b = cy.
Then for d = ua + vb

d = u(cx) + v(cy)

d = c(ux + vy)

Therefore c divides d, so c <= d.

Thus d is the gcd of a and b.

Now with Bezout's identity, we can show that the inverse of e modulo N exists when e and N are coprime:

assume $$gcd(e, N) = 1$$

By Bezout's identity, we have $$ed + kN = 1$$

Thus $$ed - 1 = -kN$$

Therefore $$ed \ \equiv \ 1 \pmod{N}$$

So finally, we know that an inverse for e exists when e is coprime with N.

----

### Making sure our ciphertext is decipherable

Note that d is unique; otherwise would mean $$ed_1 = ed_2 \ \equiv \ 1 \pmod{N}$$, which means that $$e(d_1 - d_2) \ \equiv \ 0 \pmod{N}$$, since e is coprime to N, then we can divide by e and we see $$d_1 - d_2 = 0$$, so $$d_1 = d_2$$ and we end up with a unique d anyway)

Great! But there's a further issue we need to address: we need to make sure the decryption is unique. 
This means that given a cipher c, $$c^d = m$$ should be unique. Otherwise the recipient wouldn't be able to decipher the text into a single message.
(Note of course that just because d is unique doesn't mean $$c^d \pmod{N}$$ has to be. For the trivial example, if d = N-1 (with prime N), then $$c^d \ \equiv \ 1 \pmod{N}$$ for all c)

So how do we guarantee that $$c^d$$ is unique? We need to choose an e such that $$c^d$$ has this property. And how do we do that?

Let's first introduce Euler's formula.
Euler's formula is a function $$\phi(n)$$ defined as 

$$$$\phi(n) = \text{the number of positive integers less than n that are coprime to n}$$$$

We're going to focus only on $$\phi(n)$$ for prime n. In which case, $$\phi(n) = n-1$$ trivially because of the definition of n being prime: there are no numbers less than n that divide n. 

But we're focused on N=pq, and N is not prime, so what can we do with our knowledge of $$\phi$$?

Luckily, $$\phi(N)$$ is multiplicative in this case meaning that $$\phi(pq) = \phi(p)\phi(q)$$.
To show so we leverage the so-called Chinese remainder theorem.

----

#### Chinese remainder theorem and its proof

The Chinese remainder theorem is another foundational result in number theory. The original problem dates back to the 4th century!

We'll show a weak version of the CRT here instead of the fully generalized version: If we have two numbers m and n that are coprime (gcd(m,n) = 1),
then the system $$x \ \equiv \ a \pmod{m}$$ and $$x \ \equiv \ b \pmod{n}$$ has a unique solution.

First we prove that a solution always exists.

From the first congruence: $$x = my + a$$

We can plug this into the second congruence to get $$my + a \ \equiv \ b \pmod{n}$$

Thus $$my \ \equiv \ b - a \pmod{n}$$

Because gcd(m,n) = 1, we know that m has an inverse modulo n. Let's call it m'

Then $$m'my \ \equiv \ m'(b-a) \pmod{n}$$

We can cancel m'm because we're showing congruence modulo n: $$y \ \equiv \ m'(b-a) \pmod{n}$$ (TODO can we? idk why)

Rewrite this as multiplication with remainder: $$y = zn + m'(b-a)$$

Plug this back into the first equation $$x = my + a$$: $$x = m(zn + m'(b-a)) + a$$

We can't reduce this further (because we're no longer working in modulo, so m' is not generally the inverse of m)

But we've shown that the solution for x in this system has this form

Now we prove that the solution is unique.

Assume $$x = c$$ and $$x = c'$$ both solve the system

Then c is congruent to c' mod m, ie $$c \ \equiv \ c' \pmod{m}$$

So $$c = my + c'$$, then $$c - c' = my$$

So (c - c') is divisible by m

We can repeat the same line of reasoning for n instead of m, so that (c - c') is divisible by n

Because gcd(m,n) = 1 then this means that mn also divides (c - c')

Which means $$c \ \equiv \ c' \pmod{mn}$$

In other words, x = c and x = c' implies c is unique modulo mn

----

This is important because it allows us to construct a bijection which shows that $$\phi$$ is multiplicative.

Specifically, say we have distinct primes p and q, and say we have the sets A,B,C where A is the set of integers coprime to p, B is the set of integers coprime to q, and C is the set of integers coprime to pq. 

So |A| = $$\phi(p)$$, |B| = $$\phi(q)$$, and |C| = $$\phi(pq)$$

If we can demonstrate a bijection between AxB and C, then that means that $$\phi$$ is multiplicative and we know that $$\phi(pq) = \phi(p)\phi(q)$$

The set AxB is every pair (a,b) such that gcd(a,p) = 1 and gcd(b,q) = 1 where a < p and b < q.
We take every pair (a,b) and run it through f(a,b): $$y \ \equiv \ a \pmod{p}$$ and $$y \ \equiv \ b \pmod{n}$$.
From the CRT, we know that each pair maps to exactly one solution. So we have a bijection from AxB to some set of integers D.

But we can show that D = C: if d is a solution to f(a,b) then that means d is coprime to both p and q. Since gcd(p,q) = 1 then d is also coprime to pq. 
That's our definition of the set C! So that means D = C, so we now have a bijection from AxB to C which allows us to treat $$\phi$$ as a multiplicative function.

Since p and q are both prime, we know $$\phi(pq) = (p-1)(q-1)$$.

Now we can use Euler's theorem to show $$a^{(p-1)(q-1)} \ \equiv \ 1 \pmod{pq}$$ when a is coprime with pq.

Proof:

$$$$
\begin{align}
& a^{(p-1)(q-1)} = (a^{(p-1)})^{(q-1)} \\ 
& (a^{(p-1)})^{(q-1)} \ \equiv \ 1^{(q-1)} \pmod{p} \text{(because of FLT)} \\
& \equiv \ 1 \pmod{p} \text{ (because 1 to anything = 1)} \\
\end{align}
$$$$

You can repeat same process with p and q flipped

So then $$a^{(p-1)(q-1)} \ \equiv \ 1 \pmod{p}$$ and $$a^{(p-1)(q-1)} \ \equiv \ 1 \pmod{q}$$ implies $$a^{(p-1)(q-1)} - 1$$ is divisible by both p and q, which means it's divisible by pq as well

Therefore $$a^{(p-1)(q-1)} \ \equiv \ 1 \pmod{pq}$$ as well

This was arguably a roundabout way to avoid some group theory. Instead of doing what we did: focusing specifically on $$\phi$$ for only prime numbers, using CRT to demonstrate multiplicativity of $$\phi$$, and using Euler's theorem to establish modulo behavior, we could've shown, using Lagrange's theorem, that $$\phi(n)$$ is the order of multiplicative group of modulo n. (Good luck with that lol)

Okay. Now we are *finally* ready to describe e so that we can decipher $$m^e$$ using $$c^d$$ where $$ed \ \equiv \ 1 \pmod{N}$$

Start with $$c^{d^e} = m \ \equiv \ m^{e^d} \pmod{N}$$

From theorem above, we know $$m^{\phi(N)} \ \equiv \ 1 \pmod{N}$$ (TODO remove the phi reference etc if you need to)

Take them to the kth power, $$m^{k\phi(N)} \ \equiv \ 1 \pmod{N}$$ (because 1^k = 1)

Multiply both sides by m, $$m^{(k\phi(N) + 1)} \ \equiv \ m \pmod{N}$$

$$m^{(k\phi(N) + 1)} = m^{ed} \pmod{N}$$

$$k\phi(N) + 1 = ed$$

This is important! This basically is another way of saying $$ed \ \equiv \ 1 \pmod{\phi(N)}$$
Since $$\phi$$ is multiplicative, $$\phi(N) = \phi(p)\phi(q) = (p-1)(q-1)$$
So $$ed = 1 \pmod{(p-1)(q-1)}$$
And therefore we know then that gcd(e, (p-1)(q-1)) = 1

So going back again, assume m = u is a solution to $$m^e = c \pmod{N}$$.

Because $$k(p-1)(q-1) + 1 = de$$, and since $$u = u^1$$ obviously, then we can get $$u = u^{de - k(p-1)(q-1)}$$

$$\equiv \ u^{ed} \cdot u^{((p-1)(q-1))^{-k}} \pmod{pq}$$

$$\equiv \ u^{ed} \cdot 1^{-k} \pmod{pq}$$ ; because of Euler's theorem 

$$\equiv \ c^d \pmod{pq}$$ ; because $$u^e = c$$ and $$1^{-k} = 1$$

Therefore $$c^d$$ is the unique solution when gcd(e, (p-1)(q-1)) = 1

So, p-1 and q-1 are both even numbers, and therefore e=3 satisfies the requirement. This actually gets used in practice!
But sometimes a larger e is chosen for more security (TODO remind me of the other e choice = 65167 or whatever)

This was a lot of work just to choose e=3. To recap, we needed to make sure that we could undo the original encryption.
We decrypt the ciphertext c by raising it to the dth power: $$c^d \pmod{N} \ \equiv \ m$$
But existence wasn't just enough. We also needed to make sure that decryption was unique. 
Otherwise the recipient could end up with multiple possible interpretations.

In order for $$c^d$$ to be unique modulo N, we had to choose e carefully. 
We did a lot of legwork to show that $$c^d$$ uniquely decrypts the original message when
e is coprime to (p-1)(q-1). 

### Finding the decryption key d!
TODO use Euclidean algorithm with knowledge of p and q to find d so that we can therefore decrypt m^e mod N
TODO also show why Eve can't do shit without p and q... (idk if the book actually covers this? i think it's literally an open problem)

## Practical matters
TODO start coding shit up
