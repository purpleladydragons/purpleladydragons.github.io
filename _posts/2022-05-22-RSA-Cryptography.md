- we want to exchange messages privately, but we don't have any secure channel. we have to assume someone is listening to every single message we send to each other
- if we *could* exchange at least one message in private, we could use a symmetric key system to exchange further messages privately
- but we can't. so how do we communicate at least once in an insecure channel if we can't agree on a secret key first off?
- the solution is asymmetric key systems. you can use such a system to agree on a secret key and then you can use that for symmetric exchange going forward
- why both with symmetric at all? b/c it's faster (TODO show why)

We have two major asymmetric systems: ElGamal and RSA. ElGamal is based on Diffie-Hellman key exchange, which is based on discrete logarithms. RSA is based on integer factorization.
By "based on", I mean that the security of these systems are ultimately dependent on how hard it is to solve these underlying problems. 
(And technically, the systems are possibly weaker than these underlying problems since they publicize some extra information that makes the problems slightly different)

So how does RSA work?
First off, we've noted that it will ultimately use the difficulty of integer factorization as its security measure.
That means that for an eavesdropper to decrypt a private message, it should be (roughly) as hard as factoring some number N.
Clearly, if we chose something trivial for N like 15, then an eavesdropper would very immediately be able to break our code.
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

A useful property of modular arithmetic is that exponentiation seems to generate random outputs. This is good for encryption! It means that if we put in two nearly identical messages, we get two totally different results. This means that someone monitoring all the encrypted messages, no matter to how many, should not be able to learn anything about the secret messages.
It's generally held in consensus that the outputs are truly pseudorandom and not predictable. But there's no proof yet. 

So to encrypt our message m, we will exponentiate it to somer power e, and then mod it by N: c = m^e mod N. 
(TODO why do we use N as the modulus? because uhhh the decryption algorithm relates to the modulus and not to e)

Because modular exponentiation is random, this ciphertext should be totally undecipherable by a hostile listener.
But how does the intended recipient know how to decrypt it?
Well, it turns out there's an algorithm that allows for us to decrypt it so long as we know p and q.
I will get to that algorithm in a moment, but first I want to focus on how we choose e, because we can't just choose any random number.

To decrypt the ciphertext, we want to solve for x in c = x^e mod N. If we know p and q, this is feasible. 
Essentially, we are looking to undo the encryption by inverting the exponentiation. So we are looking for some d such that c^d mod N = m. 
The first quandry is that d might not exist if we're not careful when choosing e! (TODO like... it's not invertible at all? or you get multiple results or what? ah I suppose it *is* possible since you don't control c at all, and so we don't know if c is a primitive root. it's not guaranteed, so a random e might genuinely generate some value that can't be achieved by any other... hmmm no hold on...
basically... uhhhh m^e mod N = c, and we want c^d mod N = m. You don't control c, so it's possible that there exists m such that *no* d works)
(TODO we have to explain why d might not exist better)
So how do we guarantee that d does exist for decryption purposes?
(TODO ^^ maybe re-edit above para)

Okay so this is somewhere that I really struggled. You want to find inverse d to e (basically c^d^e mod N = c mod N, so that c^d is the original message). All the literature talks about how you want de = 1 mod (p-1)(q-1). But that's not what we have here at all (for now). We have de = 1 mod pq. But we can do some quick algebra to get to de = 1 mod (p-1)(q-1). 
We have m = m^(ed) (basically decrypting an encrypted message with the right key will give us the original message. This is a crucial charateristic of our system. Without this, we wouldn't be able to decrypt the encrypted messages uniquely)
From Euler's totient function (TODO even bother going into it?) we know m^phi(N) = 1 mod N
If we take it to the kth power, then we have m^kphi = 1 mod N (b/c 1^k = 1)
And then m^(kphi + 1) = m mod N (b/c 1m = m and m^x * m = m^(x+1))
Then m^(kphi+1) = m^(ed)
So ed = kphi + 1
And this important! This is another way of saying that ed = 1 mod phi(N)
phi(N) = (p-1)(q-1) so ed = 1 mod (p-1)(q-1)
Bu thow do we actually go about finding such an e that it has an inverse d in modulus phi?
We know by prop 1.13 (TODO) then that gcd(e, (p-1)(q-1)) = 1
So boom, we now choose e such that gcd(e, (p-1)(q-1)) = 1

TODO euler's formula / theorem
formula describes function phi(n) = how many numbers up to n are coprime with n?
in the special case where n is prime, we can use FLT to show that phi(n) = n-1
but otherwise we have two options:
1) use the citations 4 and 5 to show that phi is multiplicative so that we can show phi(N) = (p-1)(q-1)
2) develop Lagrange's theorem and group theory to show that phi(n) is the order of the multiplicative group of integers mod n



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

Now that Alice has p and q, she can straightforwardly generate e as well. But why do we use e such that gcd(e, (p-1)(q-1)) = 1?
Well, assuming someone encrypts a message to her via c = m^e mod N, how would she decrypt it? She'd want to undo the exponentiation essentially,
which really means taking the inverse. So she's solving for x in x^e mod N = c. If we assume d is the inverse of e, then m = c^d.
But here's the problem: d doesn't exist unless gcd(e, (p-1)(q-1)). 

I will show you why: TODO proof of prop 1.13
 




