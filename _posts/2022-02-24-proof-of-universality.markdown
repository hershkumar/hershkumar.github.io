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

Before we dive into the different ways to look at this problem, lets first shoot off some quick definitions to make the question more understandable to someone who hasn't necessarily been exposed to a great deal of quantum computing.

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

{% raw %}
$$test$$
{% endraw %}

#### Rotation Gates
<details>
	<summary>"You can't exponentiate a matrix!"</summary>
	<p>...is probably what at least someone is thinking. Lets talk a little about how we define functions acting on matrices. This section isn't really necessary for understanding what going on, if you don't care about why we can exponentiate the matrix, feel free to just skip this section.
	Lets first see how we extend an arbitrary function \$\$f: \mathbb{R} \to \mathbb{R}\$\$ to a function that can act on a matrix in our space, \$\$f: \mathbb{C}^{2\times 2} \to \mathbb{C}^{2\times 2}\$\$</p>

</details>

### Universality


### The Task at Hand





[A neat primer on $ SO(3) $ and its finite groups for those interested.](http://math.uchicago.edu/~may/REU2020/REUPapers/Bui,An.pdf)


[^1]: Another very interesting result of this is that reversible computations [generate no heat](https://en.wikipedia.org/wiki/Landauer%27s_principle)
[^2]: Note that reversible gates must have the same number of outputs as inputs.
[^3]: While we *could* just make reversible versions of the classical gates, it turns out to be easier to just make a new set of gates *made* to be reversible. 
