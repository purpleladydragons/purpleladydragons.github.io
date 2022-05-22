- we want to exchange messages privately, but we don't have any secure channel. we have to assume someone is listening to every single message we send to each other
- if we *could* exchange at least one message in private, we could use a symmetric key system to exchange further messages privately
- but we can't. so how do we communicate at least once in an insecure channel if we can't agree on a secret key first off?
- the solution is asymmetric key systems. you can use such a system to agree on a secret key and then you can use that for symmetric exchange going forward
- why both with symmetric at all? b/c it's faster (TODO show why)

We have two major asymmetric systems: ElGamal and RSA. ElGamal is based on Diffie-Hellman key exchange, which is based on discrete logarithms. RSA is based on integer factorization.
By "based on", I mean that the security of these systems are ultimately dependent on how hard it is to solve these underlying problems. 
(And technically, the systems are possibly weaker than these underlying problems since they publicize some extra information that makes the problems slightly different)

So how does RSA work?
First off, we've noted that it will ultimately use the difficult of integer factorization as its security measure.
That means that for an eavesdropper to decrypt a private message, it should be (roughly) as hard as some number N.
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

Since we choose N, we know its two factors p and q. 

If we add another value, `e`, then we can achieve secure communication.
This is where things start to get tricky in my opinion.
We choose e such that gcd(e, (p-1)(q-1)) = 1.
Why? I'll get to that in a minute. 
But once we have this, we can publish N and e as our public key. 
Then, someone else can send us a secure message! They take their message m, and they encrypt it by calculating c = m^e mod N.
They send this value c over the wire, everyone else can see it too, but only we can read it!
That's because ultimately, the fast algorithm to decrypt c back into m relies on knowing p and q. And only we know them!

This all felt extremely unintuitive to me so let's break it into steps and I'll try to show/prove why they work. 
As for how they came up with this, well that's a lot harder and part of the reason it was such a huge accomplishment!
Probably, it started with knowing that modulo exponentiation is very unintuitive and basically random as far as we can tell. 
If you pass two numbers x and x+1 through modulo exponentiation, you get very different results (TODO not in all cases right? only under certain circumstances?)

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

Remember that modulo exponentiation is good for generating random results. m^e modulo N is going to hopefully produce a random looking output.
But we have two problems to solve before we can go ahead with that:
1. we want to make sure it really *is* hard for people that don't know p and q
2. it has to be easy for someone who knows p and q, otherwise the intended receiver can't decrypt the message either

(TODO this whole section)
ah! now I understand. e has to be prime to (p-1)(q-1) in order to generate all the results right? 
yes, that's how e is a primitive root of pq and generates all the values
otherwise, e exponenting by e would... wait fuck man
isn't it that the *base* is the root? so i don't get why e has to do this stuff...
but this is related to FLT ultimately i think

