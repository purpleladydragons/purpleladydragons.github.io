Going thru 3blue1brown calc & diff eq videos

**Differential equations tourist view**
"solving" a ODE means determining its form via what we know about its rate of change

For pendulum, the typical high school formula is actually wrong. 
It barely(?) has an analytical solution. But adding the dampening term suddenly makes it way harder - apparently there is no analytic solution (or at least it's unknown)

Talks about states. For the pendulum, the only two variables are angle and angular velocity 

<img width="1002" alt="image" src="https://github.com/user-attachments/assets/403e6934-a1d5-4c71-aa52-a78eb4e6b777" />

My understanding is this is the deterministic flow of the pendulum through state space given a starting point. 
Flows will look similar, you'll probably see this spiral shape, but in parallel or "off" parallel. 
And this likely leads us to vector fields

Yup. So each state is a vector / point in the space. You then take the derivative of the vector and this gives you which way it flows,
ie the typical vector field diagram

Oh wow. His visualization of high velocity initial states is beautiful. Because you're looking at the vector field initially and being like wow that looks wonky,
it seems like some artifact of the model that you have repeating spirals right and leftward. But nope. He shows that if you spin the thing hard enough, 
the pendulum will do a full circle before settling into its dampening oscillation. 

<img width="1061" alt="image" src="https://github.com/user-attachments/assets/a5a86e6f-dfb7-4ac9-ac0f-c4ae2b56cb6a" />

Ugh these visuals are amazing. Makes me want to write a weather simulator so badly

So this is all the phase space. In physics, generally the axes are position and momentum

Now talking about stability. Pendulum example is great: hang ball at bottom is stable. Balance ball at very top is unstable

Now he's using "love" as an example which is pretty funny. Obviously it's all in how you model it and his model isn't exactly fully realistic,
but there is something grounding about being able to model your relationship dynamics like this as a way of zooming out and making sense of things.
I know in the past, I often tried to model my life to get a forward-looking idea of where things were heading or an explanation of why I was feeling
the way I was. Idk how useful it ever was, but it at least forces you to sit and really inspect how you're behaving in a given moment and contemplating
whether you're acting along some function. And if you're _not_, that's still valuable to confirm

The larger mathy point here though is that even though the equations for the love model and the pendulum model are quite different, their phase space
behavior looks quite similar for the most part (at least within a certain region)

Solving a ODE "numerically" means just programming it basically and simulating. I imagine there is probably more nuance to it than that

Touches on chaos theory: for some class of DEs, small shifts in initial conditions lead to greater changes down the road so your simulations compound errors and stop being accurate

**What are PDEs**

He breaks heat eq down to discretized version and explains that a pt T2 will flow to the average between its neighbors. 
This can be rewritten as the diff of diffs dT3 - dT1 which can be notated as ddT1.
This discrete second difference is analogous to the second derivative!

He says that in a PDE, infinitely many variables are changing. I don't fully grok it but I guess it's the temperature at each point in space & time. Which tbh I don't fully get why that is different than the ODE case? Well I suppose in the orbit example, you have finite planets orbiting. In the heat example, you have infinitely many points along a rod to keep track of. So I guess it makes sense. 

Defines the Laplacian. Basically is just the addition of the different spatial second derivatives. Basically just the multi dimensional version asking, how much does a point differ from the average of its neighbhors? 
