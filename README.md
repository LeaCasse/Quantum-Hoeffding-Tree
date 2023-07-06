# Quantum-Hoeffding-Tree
Quantum Hoeffding Tree

\part{General idea and motivation}
\section{First idea and motivation}
Since the beginning of the internship, we try to speed-up the Hoeffding Tree algorithm by replacing the tree following lines of code by a quantum algorithm:

\begin{center}
    \includegraphics[width=0.4\textwidth]{Images/2.png}
\end{center}

Those three lines aim to return the two greatest purity gain of a list of purity gain updated each time a new sample arrived. Getting the two greatest purity gains allow to calculate how many items $n_{min}$ we should wait until splitting the leaf on the attribute $X_i$ that maximised the purity gain.\\

We choose to work with Grover because, theoretically, it is reducing the complexity by square root of the number of item in the purity gain list. Instead of having O(d) operations in a classical way, we would have O($\sqrt{d}$) instead.\

However, in practice, given that we need to encode the purity gain list into a quantum register, we already perform O(d) operation because we need to loop over all the items of the dataset so we already lost the quantum advantage. \\

\section{New idea}
This is why we choose to take the problem earlier. The idea would be to encode the purity gain dataset into the amplitudes of a quantum state (qubits of a QuantumRegister) and to update these each time a new item arrived. No need to keep the previous list because we only need the last updated one. At the end, we would only need to measure the state to have access to the indices of the two greatest purity gain in the updated list.\\

The first motivation come from the fact that we could convert the function to calculate purity gain into a quantum circuit that could allow to calculate in superposition all the purity gains. \ 

The second motivation is due to the fact that we do not need to encode the dataset (because we can start with all the purity gain set to 0) so we do not have a big complexity because of the classical data encoding.

\section{How?}

The goal would be to have a quantum circuit allowing to calculate simultaneously (thanks to the quantum superposition principle) all the gain values $G(X_i)$ of each attribute each time a new item $(x^n, y^n)$ arrives in the stream.\\

In fact, we only need to input the $n_j$ and $n_{jk}$ populations that are updated classically as soon as a new item arrives in the stream. Indeed these populations are lists of lists that grow as items arrive in the stream. \ 

The goal of the quantum algo would be to take as input these two populations, to calculate the new list $(G(X_1), ..., G(X_i), ..., G(X_d))$ based on these updated populations and to return counts to identify indices a and b of the two maximum gain values $G(X_a)$ and $G(X_b)$. We can then classically calculate these two values (because we will know the indices a and b of these two max values but not directly their values $G(X_a)$ and $G(X_b)$) and then classically calculate the Hoeffding statistic thanks to these two values. \ 

From there we no longer need the list of gains since a new item arrives in the stream, we update the new populations $n_j$ and $n_{jk}$ then we can again use the quantum algorithm to recalculate the new list of gains $(G(X_1), ..., G(X_i), ..., G(X_d))$, measure the quantum state to obtain the counts and thus the indices of the two new maximum values $G(X_a)$ and $G(X_b)$ to again calculate the Hoeffding statistics classically and so on. 

\subsection{Gain function}

The purity gain function is defined as :

\begin{equation}
    \hat{G_{l_m}}(X_i)= \sum_{j=1}^{v} \hat{P}(l_j)\hat{Ent}(l_j)
\end{equation}

Given that the populations $n(l_j)$ (number of samples in $l_j$, that is to say, the number of sample in $l_m$ with $X_i=x_{ij}$) and $n_k (l_j)$ (number of sample $(x^n,y^n)$ in the leave $l_j$ which is labelled with $y^n=y_k$) are already update when a new item arrived, we only need those to update the purity gain list.\\ 

Let's define the purity gain as a function of these two populations:

\begin{equation}
    \hat{G_{l_m}}(X_i)= \sum_{j=1}^{v} \frac{n(l_j)}{n(l_m)} * ( - \sum_{k=1}^{c} \hat{P_k}(l_j) ~ log(\hat{P_k}(l_j)))
\end{equation}

\begin{equation}
    \hat{G_{l_m}}(X_i)= \sum_{j=1}^{v} \frac{n(l_j)}{n(l_m)} \times ( - \sum_{k=1}^{c}  \frac{n_k(l_j)}{n(l_j)} ~ log( \frac{n_k(l_j)}{n(l_j)}))
\end{equation}

\begin{equation}
    \hat{G_{l_m}}(X_i)= - \sum_{j=1}^{v} ~ \sum_{k=1}^{c} \frac{n(l_j)}{n(l_m)} \  \frac{n_k(l_j)}{n(l_j)} ~ log( \frac{n_k(l_j)}{n(l_j)})
\end{equation}
 ($\hat{G_{l_m}}(X_i)$ still depend of i but it is implicit in the function definition: index j depend on i).
 
\subsection{Classical function convert into a quantum circuit}

To implement the classical function to calculate the purity gain into a quantum circuit, I wonder that we have many options:

\begin{itemize}
    \item $ClassicalFunction()$ : The classical function compiler provides the necessary tools to map a classical irreversible functions into quantum circuits. Below is a simple example of how to synthesize a simple boolean function defined using Python into a QuantumCircuit
    \item $classical\_to\_quantum()$: github Quantastica 
\end{itemize}

\newpage
\part{Classical Hoeffding Tree}

\section{HT Pseudo code}

\begin{center}
    \includegraphics[width=0.5\textwidth]{Images/3.png}
\end{center}

\section{General case}

In this section, I explain the idea of the Hoeffding Tree (HT) with my words. First of all, this is the notations I use:

\begin{itemize}
    \item n to index each item/sample/example/instance of the data stream S such as $S=\{(x^1,y^1),(x^2,y^2),...,(x^N,y^N)\}$ and $n \in [1;N]$
    \item i to index the attribute/feature such as : $X=(X_1,...,X_d)$ and $i \in [1;d]$
    \item j to index the attribute value such as: $X_i \in \{ x_{i1},...,x_{ij},...,x_{iv} \} $ and $j \in [1;v]$
    \item k to index the class label such as: $y^n \in \{ y_{1},...,y_{k},...,y_{c} \}$ and $k \in [1;c]$ 
    \item m to index each nodes (leaves to be split) such as: $l_m \in \{l_1,...,l_m,...,l_l \}$ and $m \in [1;l]$
\end{itemize}

If each sample would be a football player then the stream S would be : $S=\{$ (Yoann, Good player),(Sandra, Good player),...,(Lorenzo, Bad player $)\}$. Each player would be defined by a list of attributes so X would be : X = (Height, Age, Weight,..., Sex). Then, each attribute would have it sub set of possible values so, for the attribute $X_i$ = Age, the values $x_{ij} = age_j$ could be $\{ 0, 1, ..., 100 \}$. This sub set is different for each attribute. Finally, the class could only take two values: $y^n \in \{ $Good player, Bad player $\}$.\\

The HT is incrementally built by finding the attribute $X_i$ which give the best purity gain $\hat{G_{l_m}}(X_i)$ for the split of the leave $l_m$. 


Here, $\hat{G_{l_m}}(X_i)$ equals to :

\begin{equation}
    \hat{G_{l_m}}(X_i)= |\sum_{j=1}^{v} \hat{P}(l_j)\hat{Ent}(l_j)-\hat{Ent}(l_m)|
\end{equation}

With $\hat{P}(l_j)$ is the estimation of the probability of $X_i$ to be in $l_j$ (that is to say the probability of having a sample which has it attribute $X_i$ equals to $x_{ij}$ in the leaf $l_j$):

\begin{equation}
    \hat{P}(l_j) = \frac{n(l_j)}{n(l_m)}
\end{equation}

And with $\hat{Ent}(l_m)$ the entropy value of $l_m$. We could have use the Gini metric as well. This is how there are defined:

\begin{equation}
    \hat{Ent}(l_m)= - \sum_{k=1}^{c} \hat{P_k}(l_m) ~ log(\hat{P_k}(l_m))
\end{equation}
\begin{equation}
    \hat{Gini}(l_m)= 1 - \sum_{k=1}^{c} (\hat{P_k}(l_m))^2
\end{equation}

These metrics give us the purity of a state so when the result is near 0 so the result is purer than when the metric result is near to 1.\\

With $\hat{P_k}(l_m)$ the probability to have a sample $(x^n,y^n)$ in the leave $l_m$ which is labelled with $y^n=y_k$:

\begin{equation}
    \hat{P_k}(l_m) = \frac{n_k(l_m)}{n(l_m)}
\end{equation}

With : 

\begin{itemize}
    \item $n_k(l_m)$ the number of sample $(x^n,y^n)$ in the leave $l_m$ which is labelled with $y^n=y_k$.
    \item $n(l_m)$ the number of samples in $l_m$
\end{itemize}

We need to wait for a certain number $n_{min}$ of data samples to arrive $ \{ (X^{(1)},Y^{(1)}), … ,(X^{(n_{min})},Y^{(n_{min})}) \} $ to have a good estimation of the gain. The question then becomes whether the gain calculated on this small base is correct. For this we choose a confidence parameter $\delta$ which allows us to write the following concentration inequality: 

\begin{equation}
\label{f}
    P [ | \hat{G}(X_i) - E[\hat{G}(X_i)]| \leq \epsilon (n_{min},\delta)] \geq 1-\delta
\end{equation}

With $lim_{n_{min} \longrightarrow \infty}\epsilon = 0$ and $\epsilon$ which is called the Hoeffding Bound and is equals to :

\begin{equation}
\label{eps}
    \epsilon = \sqrt{\frac{R^2 ln(\frac{1}{\delta})}{2n}}
\end{equation}

With:
\begin{itemize}
    \item R is the range of a random variable. For a probability the range is 1, and for an information gain the range is log c, where c is the number of classes.
    \item $\delta$ is the confidence. 1 minus the desired probability of choosing the correct attribute at any given node.
\end{itemize}
So, if our confidence parameter $\delta$ is good enough we can state :

\begin{equation}
    E[\hat{G}(X_i)] \geq \hat{G}(X_i) - \epsilon (n,\delta)
\end{equation}

Now the goal is to find the $X_i$ such that $E[\hat{G}(X_i)]$ is maximum. For this, we have to wait for an optimal number of samples $n_{min}$ and have a sufficient confidence parameter $\epsilon (n,\delta)$ so that the intervals are small enough so that we can compare all the $E[\hat{G}(X_i)]$ .

\section{Simple and concrete example}
Let's take a simple and concrete example. Let's take S as:

\begin{equation}
    S = \{ ([2,3,1],2),([1,2,1],1),([1,1,2],2),([2,3,2],2),([2,1,1],1) \}
\end{equation}

Here, we choose to represent the data $x^n$ with 3 attributes $X_i$ such as:

\begin{itemize}
    \item $X_1=\{1,2\}$
    \item $X_2=\{1,2,3\}$
    \item $X_3=\{1,2\}$
\end{itemize}

 Let's consider the state where we split the root with the attribute $X_1$ and now we are in the leave l where the attribute $X_1=2$. We currently still have 2 attributes on the three to split the leave l. So, we need to calculate $\hat{G_l}(X_2)$ and $\hat{G_l}(X_3)$. To do that, we'll need to calculate for each gain, the entropy value of the current leave l and the 2 (for $\hat{G_l}(X_2)$ ) or 3 (for $\hat{G_l}(X_3)$ ) new leaves. For example, let's calculate the three entropies needed to obtain the gain $\hat{G_l}(X_3)$:

 \begin{itemize}
     \item $\hat{Ent}(l) = \frac{n_1(l)}{n(l)} log(\frac{n_1(l)}{n(l)}) + \frac{n_2(l)}{n(l)} log(\frac{n_2(l)}{n(l)})$
     \item $\hat{Ent}(l_1) = \frac{n_1(l_1)}{n(l_1)} log(\frac{n_1(l_1)}{n(l_1)}) + \frac{n_2(l_1)}{n(l_1)} log(\frac{n_2(l_1)}{n(l_1)})$
     \item $\hat{Ent}(l_2) = \frac{n_1(l_2)}{n(l_2)} log(\frac{n_1(l_2)}{n(l_2)}) + \frac{n_2(l_2)}{n(l_2)} log(\frac{n_2(l_2)}{n(l_2)})$
 \end{itemize}

In our case, we have 3 samples in l because there are 3 samples which have $X_1=2$ so $n(l)=3$, 1 sample with $y_k=1$ in l so $n_1(l)=1$ and 2 samples with $y_k=2$ so $n_2(l)=2$.\ 

Then, we have 2 samples in $l_1$ because there are 2 samples which have $X_1=2$ and $X_3=1$ so $n(l_1)=2$, 1 sample with $y_k=1$ in l so $n_1(l_1)=1$ and 1 sample with $y_k=2$ so $n_2(l_1)=1$.\

Finally, we have 1 samples in $l_2$ because there is only one sample which has $X_1=2$ and $X_3=2$ so $n(l_2)=1$, 0 sample with $y_k=1$ in l so $n_1(l_1)=0$ and 1 sample with $y_k=2$ so $n_2(l_1)=1$.\\

Now, we obtain these entropies:

 \begin{itemize}
     \item $-\hat{Ent}(l) = \frac{1}{3}log(\frac{1}{3}) + \frac{2}{3}log(\frac{2}{3}) = 0.30+1.21 = 1.51$ 
     \item $-\hat{Ent}(l_1) = \frac{1}{2}log(\frac{1}{2}) + \frac{1}{2}log(\frac{1}{2}) = 0.72+0.72 = 1.44$
     \item $-\hat{Ent}(l_2) = \frac{0}{1}log(\frac{0}{1}) + \frac{1}{1}log(\frac{1}{1}) = 0+0 = 0 $
 \end{itemize}

We can now calculate $\hat{G_l}(X_3)$:

\begin{equation}
    \hat{G_l}(X_3) = |\frac{n(l_1)}{n(l)}*-\hat{Ent}(l_1)+\frac{n(l_2)}{n(l)}*-\hat{Ent}(l_2) - (-\hat{Ent}(l))|
\end{equation}
\begin{equation*}
    \implies \hat{G_l}(X_3) = |\frac{2}{3}*-1.44+\frac{1}{3}*-0 - (-1.51)| = 0.07
\end{equation*}

Let's now choose $\delta=0.01$ and $\epsilon=0.014$. Given that there are both functions of the number of samples $n_{min}$ (cf \ref{eps}), we now find the number of samples $n_{min}$ we need to wait to obtain the following in-equation to be respect :

\begin{equation}
    \label{bound}
    P[|\hat{G_l}(X_3)-E\hat{G_l}(X_3)| \leq \epsilon(n_{min}0 ] \geq 1- \delta(n_{min})
\end{equation}
