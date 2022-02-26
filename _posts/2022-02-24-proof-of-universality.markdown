---
layout: post
title:  "(Dis)proof of Universality of H + S"
date:   2022-02-24 18:11:01 -0400
categories: latex, quantum-computing
usemathjax: true
published: false
---

On my latest PHYS457 problem set, we were tasked with solving a bonus problem that seemed pretty innocuous but turned into a massive timesink, and was easily the most challenging problem on the entire set. In this post, I want to go over what the problem was, a little bit of background on how to interpret it, and then the (various) methods that we used to try and solve it. I'll cap the post off by talking a little bit about why I find this problem interesting and how it's an important consideration for near-term quantum computing.

### The Problem

The problem statement itself is, to be completely honest, a bit underwhelming at first glance. We are given the task of **proving whether or not the set of gates $\\{CNOT, H,T^2\\}$ is universal**. A neat little one-liner right? Thats what me from 7 hours of work ago thought too.

Before we dive into the different ways to look at this problem, lets first delve into some of the basics of quantum information, for those who are new to the subject.

### Quantum Gates
Since near-term quantum computers are restricted in the number of qubits that are accessible, we deal with quantum computation at the most basic level, logic gates. Now while you may be familiar with classical boolean logic (operations such as $\texttt{AND}$, $\texttt{OR}$, $\texttt{NOT}$, and $\texttt{XOR}$), quantum logic gates are subject to a constraint that classical logic gates are not. Due to the postulates of quantum mechanics, specifically the postulate that states that the time-evolution of a system is based off of the Hamiltonian $\hat{H}$ of the system and the Schrodinger equation, we have a requirement that all quantum logic operations be *reversible*. Now the definition of reversible means that given the output of a logical operation, we can obtain the set of inputs that must have been provided in order to obtain the output[^1]. This concept is made easier to understand through the use of some simple examples. 

Suppose we have the classical $\texttt{AND}$ gate, which takes in two bits of input, and outputs a bit that is $1$ if and only if the two input bits are both $1$. Now let's say that we are told that the output of an $\texttt{AND}$ operation is $0$. We know that the inputs to the gate are two bits $a$ and $b$. From the definition of the $\texttt{AND}$ gate, we know that $a \neq 1$ and $b \neq 1$. However, we cannot determine the exact values of $a$ and $b$, because we have 3 possibilities for the inputs that would return $0$, namely $(a,b) \in \\{(0,0), (1,0), (0,1)\\} $. We see that this classical gate is **not** reversible, we cannot always recover the inputs from the outputs. For you math nerds, we can see that the operation is a function that obeys the mapping $f: \\{0,1\\}^2 \to \\{0,1\\}$.

Now on the other hand, let us define our first quantum gate, the $\texttt{CNOT}$ gate. This gate is a 2-qubit gate, it takes in 2 qubits as input, and returns 2 qubits as output[^2]. The $\texttt{CNOT}$ has a control qubit, and a target qubit. At a high level, the $\texttt{CNOT}$ flips the state of the target qubit if the first qubit is $0$. We can think of this as the same operation as the classical $\texttt{XOR}$ gate, except instead of returning only 1 output, we store the result of the $\texttt{XOR}$ in the second qubit. This may be a little hard to visualize, so schematically:

\$\$ \texttt{CNOT}(a,b) \to (a, a\oplus b) \$\$

Which can be compared to the classical $\texttt{XOR}$:

\$\$\texttt{XOR}(a,b) \to (a\oplus b)\$\$

After a little bit of thinking, we can see that the $\texttt{XOR}$ gate is not reversible, while the $\texttt{CNOT}$ gate is.

Okay so quantum gates have to be reversible, so what? Well the bad news is that we have to dump all the classical gates in the garbage[^3]. We have to define a whole new set of gates specifically for quantum computation, each of which is a reversible process. Before we jump right into talking about quantum gates, lets do a little detour to talk about how we represent these gates, and how we can represent the states that they modify.

#### State Spaces and Matrix Representations
One of the 6 postulates of quantum mechanics is that our entire system lives in a special vector space known as a Hilbert Space. Now without getting into the weeds, lets just say that we're in a vector space that has some special properties (none of which we'll need in particular here). This means that we can represent any possible quantum system as some vector in our space, specifically as some linear combination of the basis vectors. Now those familiar will linear algebra will note that we have an infinite number of basis sets for our vector space. While this is true, we generally represent states in what is known as the computational basis, $ \\{|0\rangle, |1\rangle\\}$. This means the state of any qubit in our system is given by:

\$\$\|\psi\rangle = \alpha \|0\rangle + \beta \| 1\rangle\$\$

Where we have used what physicists call Dirac notation to represent the vectors. Representing them in a way that will probably make more sense to someone versed in linear algebra:

\$\$ \begin{pmatrix}\alpha \\\\ \beta \end{pmatrix} = \alpha \begin{pmatrix}1 \\\\0\end{pmatrix} + \beta \begin{pmatrix}0\\\\1\end{pmatrix}\$\$

Where we have implicitly defined the conventional matrix representation of the basis vectors, $ \| 0 \rangle = \begin{pmatrix}1\\\\0\end{pmatrix}, \| 1 \rangle = \begin{pmatrix}0\\\\1\end{pmatrix}$. When we think of our system in this way, we can see the intuition behind thinking that matrices are going to be how we interact with our system, since that's how we think about messing around with vectors in a vector space. This leads us to the fact that any possible quantum logic gate for our 1-qubit system can be represented by some $2\times 2$ matrix. However, not just any matrices can be used as gates, we have the condition that the matrix must be [unitary](https://en.wikipedia.org/wiki/Unitary_matrix). 

Let me now introduce some of the important gates that we care about, expressed in their matrix forms:

\$\$ X = \begin{pmatrix}0 & 1 \\\\ 1 & 0\end{pmatrix} \quad Y = \begin{pmatrix}0 & -i \\\\ i & 0\end{pmatrix} \quad Z = \begin{pmatrix}1 & 0 \\\\ 0 & -1\end{pmatrix}\$\$

These are known as the Pauli matrices, and are very important in quantum information[^4]. And now to introduce the matrices that are central to the problem, $H$ and $T$:

\$\$ H = \frac{1}{\sqrt{2}}\begin{pmatrix}1 & 1 \\\\ 1 & -1\end{pmatrix} \quad T = \begin{pmatrix}1 & 0 \\\\ 0 & e^{i\frac{\pi}{4}}\end{pmatrix}\$\$

$H$ is known as the Hadamard gate, and is central in quantum computation because of its ability to generate superpositions, $H \|0 \rangle = \frac{1}{\sqrt{2}}(\|0\rangle + \|1\rangle)$. The $T$ gate is a little bit harder to understand, but it can be thought of a gate that adds a *phase* to a qubit (a phase of $e^{i\frac{\pi}{8}}$ to be exact). What phase is exactly is honestly not too important, but it can be thought of as being some property of the state of the qubit that can be modified.

Now let's talk about a way of thinking about gates and how they act on our qubits.
#### Gates as Rotations
The basic idea with thinking of gates as rotations is that *every 1-qubit quantum logic gate can be thought of as a rotation by some angle $\theta$ along some axis $\hat{n} = (n_x,n_y,n_z)$ in 3D space.*  This 3D space that we think about is a visualization known as the Bloch Sphere:

![image](/images/bloch.png)

Any state of our system can be written as
\$\$ \|\psi\rangle = \cos\frac{\theta}{2} \| 0 \rangle + e^{i\phi}\sin\frac{\theta}{2} \| 1\rangle \$\$

For some angles $\theta$ and $\phi$. 

When we think about rotations, we have 3 principle axes, $x$, $y$, and $z$, as seen on the Bloch sphere. It turns out that the 3 Pauli matrices act as rotations of $\theta = \pi$ along the principal axes of the sphere, that is, the Pauli $X$ gate rotates along the $x$ axis by $\pi$, and likewise for the $Y$ and $Z$ gates. Using these, we can actually define rotations along these principal axes with arbitrary angle:
\$\$ R_x(\theta) = e^{i\frac{\theta}{2}X} \quad R_y(\theta) = e^{i\frac{\theta}{2}Y} \quad R_z(\theta) = e^{i\frac{\theta}{2}Z}\$\$

I won't go through the math here, but it isn't too hard to verify that these indeed rotate our state by the angle $\theta$ along the correct axis.

#### You wouldn't exponentiate a matrix!
...is probably what at least one person is thinking. Lets talk a little about how we define functions acting on matrices. This section isn't really necessary for understanding what going on, if you don't care about why we can exponentiate the matrix, feel free to just skip this section.

Lets first see how we extend an arbitrary function \$f: \mathbb{R} \to \mathbb{R}\$ to a function that can act on a matrix in our space, \$f: \mathbb{C}^{2\times 2} \to \mathbb{C}^{2\times 2}\$. There are actually two ways to extend the definition to matrices, but I'm going to show the fun one, because the other one involves series expansions and nobody likes those. Take my word for it, the two methods give the same definition of a function on a matrix.

Suppose we have some matrix $A$, which satisfies the property $A=A^\dagger$. If we have some $f: \mathbb{R}\to \mathbb{R}$, we want to define $f(A)$. Our neat trick is to exploit the [spectral decomposition](https://en.wikipedia.org/wiki/Eigendecomposition_of_a_matrix) of $A$:

\$\$A = \sum \lambda_i \|\psi_i\rangle \langle \psi_i \|\$\$

Where $\lambda_i$ is the $i$th eigenvalue of $A$ and $\|\psi_i\rangle$ are the eigenvectors. We define the action of $f(A)$ to be

\$\$f(A) = \sum f(\lambda_i) \|\psi_i\rangle \langle \psi_i \|\$\$
We use this as our definition, and this allows for some nice properties to emerge, especially when talking about exponentiation.

### Universality
Alright, so we've now defined these quantum gates, but why'd we have to go through all that? Well before we can really tackle the gist of the problem, we have to talk about universality. The notion of universality is really quite simple. A set of 1-qubit quantum gates is defined to be universal iff **using those gates, we can approximate any unitary gate acting on one qubit**. This is actually pretty intuitive, we're just saying that we can make any gate out of the gates we have in our set. This is actually an extension of the classical definition of universality, which is a well-studied topic in classical computation. However is is more difficult to find quantum universal sets, as intuitively there are a finite set of possible classical gates (large yes, but necessarily finite), while there is an infinite set of quantum gates. 

So what does universality have to do with representing these rotations as matrices? Well, we can put the definition of universality together with the fact that *every* gate can be represented using a rotation, and claim that we have universality if and only if the set of gates we use can achieve every single rotation along the Bloch sphere. In group theory terms, we need our set to *densely populate* the sphere. If it turns out that the set of rotations generated using our gate set is finite, we can conclude that the gate set is not universal, because there are some gates that our gate set cannot generate. 

### The Task at Hand
And now we finally arrive at the main attraction (hope I haven't bored you to death with definitions), the bonus question on my homework. We are tasked with determining whether the gate set $\\{\texttt{CNOT}, H, T^2\\}$ is universal.

Alright, so where do we start with this problem? Well the first thing that we do is drop the $\texttt{CNOT}$ gate in the gutter. Not actually, but at least for the vast majority of this problem, we can disregard the gate entirely, because the operations it provides are only revelant when looking at 2 qubits. In fact, it is general true that $\texttt{CNOT}$ along with any set of gates that are universal for 1-qubit operations is a universal gate set. This means that we just need to see if $\\{H,T^2\\}$ is universal.

Alright, so we've pretty easily dropped the size of our gate set, but the real hard part comes when we have to prove that we can generate any 1-qubit gate with just $H$ and $T^2$. While the idea of universality might be pretty intuitive, proving it can be very slippery. Let's go through some of the failed ideas on how to prove that this set is in fact, **not universal**.

#### Doing Some Math
The first idea that we had was to use a neat little fact, that any rotation on the Bloch sphere can be "decomposed" so to speak, using the $X$, $Y$, and $Z$ gates. Suppose we have a gate $\hat{U}$ that rotates along the axis given by $(n_x,n_y,n_z) \in \mathbb{R}^3$, by the angle $\theta$. It can be shown[^5] that we can represent this gate as

\$\$ \hat{U} = e^{i\phi}e^{-i\frac{\theta}{2}(n_xX + n_yY + n_zZ)}\$\$

Now let us use this fact to write out the exponential forms of the $H$, $T$, and $T^2$ gates:

\$\$ H = e^{i\frac{\pi}{2}}e^{-i\frac{\pi}{2}\frac{1}{\sqrt{2}}(X+Z)}\$\$
\$\$ T = e^{-i\frac{\pi}{8}}\$\$
\$\$ T^2 = S = e^{-i\frac{\pi}{4}} \$\$

Where we note that convention is to call the $T^2$ gate the $S$ gate. Now let us make an observation about the general form of a combination of the two gates. Any gate that can be constructied from our two gates will be of the form
\$\$\hat{U} = H^{a_1} S^{b_1}H^{a_2}S^{b_2}\dots\$\$

Now we can begin to retrace our first attempt at determining the universality. In order to disprove universality, we need just one unitary gate that cannot be made by any combination of $H$ and $S$. So via a little bit of intuition, we choose the $T$ gate as our target gate. We want to prove that the $T$ gate cannot be constructed by any combination of $H$ and $S$ gates.

Let us first assume that the $T$ gate *can* be made from the two gates. This means that it can be written in the form

\$\$\ T =  H^{a_1} S^{b_1}H^{a_2}S^{b_2}\dots $\$
for some $a_1, b_1, a_2,b_2,\dots \in \mathbb{Z}$. We now replace every occurrence of $H$ with its exponential form, and do the same for $T$ and every occurence of $S$. Note that we discard the external phase factor on the $H$ representation, because global phase factors do not affect the state. This leaves us with something of the form
\$\$e^{-i\frac{\pi}{8}Z} = e^{-in_H\frac{\pi}{2}\frac{1}{\sqrt{2}}(X+Z) - in_S\frac{\pi}{4}Z}\$\$

Where $n_H = \sum a_i$ and $n_S = \sum b_i$. Note that these are integers. We now do a little bit of algebra, combining the terms that have $Z$s in them:
\$\$ e^{-i\frac{\pi}{8}Z}  = e^{-i\left[\left(\frac{\pi}{2\sqrt{2}}n_H + \frac{\pi}{4}n_S\right)Z + \frac{\pi}{2\sqrt{2}}n_H X\right]}\$\$
Now comparing the left and the right side, we need 
$$\frac{\pi}{2\sqrt{2}}n_H = w\pi$$
where $w \in \mathbb{Z}$, in order to match the left and right sides of the equation. If we look at this, we essentially have that an integer must be a multiple of another integer, but that multiple is irrational. Clearly this is not possible, and thus we have reached a contradiction, and $T$ cannot be generated using $H$ and $S$. So... we're done here? We can pack it up and head on home? 

Not quite unfortunately. Astute linear algebraists may have spotted our mistake, which is that $e^Ae^B = e^{A+B}$ only holds true if $AB = BA$, that is, the matrices commute. Unfortunately, the Pauli matrices do not commute[^6], leaving us in a bit of a pickle. Now we could use something like the [Baker-Campbell-Hausdorff formula](https://en.wikipedia.org/wiki/Baker%E2%80%93Campbell%E2%80%93Hausdorff_formula) to essentially add in correction terms for the commutators, but uh... looking at the formula gives me a headache so we decided to pass on that one.


#### Further Reading
[A neat primer on $ SO(3) $ and its finite groups for those interested.](http://math.uchicago.edu/~may/REU2020/REUPapers/Bui,An.pdf)


[^1]: Another very interesting result of this is that reversible computations [generate no heat.](https://en.wikipedia.org/wiki/Landauer%27s_principle)
[^2]: Note that reversible gates must have the same number of outputs as inputs.
[^3]: While we *could* just make reversible versions of the classical gates, it turns out to be easier to just make a new set of gates *made* to be reversible. 
[^4]: Not just quantum information, any matrix $A \in \mathbb{C}^{2\times 2}$ can be represented as a linear combination of scalar multiples of the Pauli matrices.
[^5]: This was actually one of the other problems on the homework.
[^6]: The commutation relations of the Pauli matrices can be found [here](https://en.wikipedia.org/wiki/Pauli_matrices#Commutation_relations).
