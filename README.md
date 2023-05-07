Download Link: https://assignmentchef.com/product/solved-eee554-midterm-project-the-ziggurat-algorithm
<br>
Have you ever wondered how Matlab generates Gaussian random numbers? The method it uses, called the Ziggurat algorithm, is the topic of this midterm project. Each of you will implement the Ziggurat algorithm for a different continuous distribution. (See the end of this document for your assigned distribution.)

First, read and understand this document explaining the algorithm. Then write code implementing it. (As usual, Matlab is recommended but not required.) In addition, the project requires you to do various analytical tasks to correctly set up the algorithm and analyze it. You should turn in a report which includes your well-documented code, as well as a write-up describing your implementation details, and the answers to specific questions asked below. Specific tasks that <em>must </em>be covered in your project report will be labeled by the bold word Task.

Your grade for the midterm project will be 50% based on the correctness of your algorithm implementation, and 50% based on the clarity of your project report. You may discuss the project in broad terms with other students, but you must work independently.

<h1>1          Ziggurat algorithm overview</h1>

The Ziggurat algorithm is a version of rejection sampling, optimized in a way so that some significant pre-computation is required, but once this pre-computation is done, generating each sample from the distribution is, on average, extremely fast. For example, Matlab can generate Gaussian random numbers almost as fast as uniform random numbers, even though the Gaussian distribution is much more complicated.

As described in Homework 4 problem 3, the idea of rejection sampling is to define a twodimensional region R that contains the target PDF <em>f<sub>S</sub></em>. This idea can be generalized slightly by using a function <em>g</em>(<em>x</em>) with is proportional to the target PDF <em>f<sub>S</sub></em>(<em>x</em>); that is, <em>f<sub>S</sub></em>(<em>x</em>) = <em>cg</em>(<em>x</em>) for some constant <em>c</em>. Then the region R should be such that it includes the function <em>g</em>(<em>x</em>). (See Figure 1.) A random pair <em>X,Y </em>is generated uniformly in the region R. If <em>Y &lt; g</em>(<em>X</em>), then the sample <em>X </em>is accepted. Otherwise the sample is rejected and another random pair is generated, repeating until a sample is accepted. To make rejection sampling efficient, two objectives must be satisfied:

<ol>

 <li>The region R should be designed so that it is easy to generate random pairs <em>X,Y </em></li>

 <li>The probability of accepting a given pair should be as large as possible. This requires that theregion R does not include much area above <em>g</em>(<em>x</em>).</li>

</ol>

The Ziggurat algorithm assumes that the function <em>g</em>(<em>x</em>) is monotonically decreasing. The region R consists of <em>n </em>sub-regions stacked on top of each other: <em>n</em>−1 of these sub-regions are rectangles, and the <em>n</em>th is rectangle adjoined to the tail of the distribution. This idea is illustrated in Figure 2, where each of the <em>n </em>regions (in the figure <em>n </em>= 4) is shaded in a different color.

The rectangles are set up so that each of the <em>n </em>sub-regions (including the <em>n</em>th region, which is not a rectangle) have exactly the same area. Thus, a point <em>X,Y </em>can be uniformly chosen from the full region as follows: First, choose a sub-region from a discrete uniform distribution—i.e.,

Figure 1: The basic rejection sampling concept.

Figure 2: Diagram of the Ziggurat algorithm with <em>n </em>= 4. The name “Ziggurat algorithm” comes from the fact that this picture looks a little bit like a Ziggurat.

each with probability 1<em>/n</em>—in Matlab this can be done with the function randi. Then, for the rectangular regions, <em>X </em>can be chosen from a uniform distribution along the length of the rectangle, and <em>Y </em>can be chosen from a uniform distribution along the height of the rectangle. The <em>n</em>th region requires a more complicated algorithm, but since this is only needed with probability 1<em>/n</em>, this more complicated algorithm only needs to be run rarely. Thus, choosing a candidate <em>X,Y </em>is easy, which satisfies objective 1. If the number of regions <em>n </em>is large enough, then the stack of regions closely matches the function <em>g</em>(<em>x</em>), which satisfies objective 2.

If we focus on a single one of the rectangular regions, we can see another feature that makes the Ziggurat algorithm so fast. Figure 3 shows the <em>k</em>th region, for some <em>k &lt; n</em>. The <em>k</em>th region consists of a rectangle extending horizontally from <em>x</em><sub>1 </sub>to <em>x<sub>k</sub></em><sub>+1</sub>, and vertically from <em>y<sub>k</sub></em><sub>+1 </sub>to <em>y<sub>k</sub></em>. As shown in Figure 2, <em>y<sub>k </sub></em>= <em>g</em>(<em>x<sub>k</sub></em>) for each <em>k</em>. Since <em>g</em>(<em>x</em>) is decreasing, <em>y<sub>k</sub></em><sub>+1 </sub><em>&lt; y<sub>k</sub></em>. We can easily choose a point uniformly from this rectangle by generating <em>X </em>∼ U(<em>x</em><sub>1</sub><em>,x<sub>k</sub></em><sub>+1</sub>) and <em>Y </em>∼ U(<em>y<sub>k</sub></em><sub>+1</sub><em>,y<sub>k</sub></em>). After generating this point, we need to check whether <em>Y &lt; g</em>(<em>X</em>). However, if <em>X &lt; x<sub>k</sub></em>, then <em>Y &lt; g</em>(<em>X</em>) no matter what <em>Y </em>is! This means that we do not even need to generate <em>Y </em>, nor evaluate the function <em>g</em>(<em>X</em>), unless <em>X &gt; x<sub>k</sub></em>. If <em>n </em>is chosen to be large enough, each rectangle will be much wider than it is tall, so the case where <em>X &gt; x<sub>k </sub></em>occurs only rarely.

To implement the Ziggurat algorithm, first the values <em>x</em><sub>1</sub><em>,x</em><sub>2</sub><em>,…,x<sub>n </sub></em>and <em>y</em><sub>1</sub><em>,y</em><sub>2</sub><em>,…,y<sub>n </sub></em>must be pre-computed. This pre-computation requires some effort, but it only needs to be done once. After

Figure 3: Diagram of the <em>k</em>th region of the Ziggurat algorithm.

these values are computed, the algorithm to generate one sample from the target distribution is summarized by the following pseudo-code.

The above pseudo-code does not give details about how do deal with the <em>n</em>th region. We’ll get back to that! Note that, most of the time, to generate a sample of the target distribution, all we need to do is generate one uniform discrete variable <em>K</em>, generate one continuous uniform random variable <em>X</em>, and do one comparison <em>X &lt; x<sub>k</sub></em>. Even though the alternative paths—when <em>k </em>= <em>n</em>, or <em>X &gt; x<sub>k</sub></em>, or <em>Y &gt; g</em>(<em>X</em>)—are more complicated, these alternative paths are rare, so most of the time we do not need to execute them.

<h1>2          Implement a baseline sampling algorithm</h1>

Each of you is assigned a continuous distribution with a PDF of the form

where <em>g</em>(<em>x</em>) is a monotonically decreasing function in the range <em>x &gt; x</em><sub>1</sub>. See Section 6 for your distribution assignments.

Task 1 <em>Find the constant </em><em>c so that your PDF is normalized correctly.</em>

Calculating <em>c </em>requires evaluating the integral

Z ∞

<em>g</em>(<em>x</em>)<em>dx.</em>

<em>x</em>1

For some distributions, this integral is solvable in closed form. If it is not, you may use numerical integration software to calculate <em>c </em>numerically. (In Matlab, you should use the integral function.)

Task 2 <em>Write a function that computes the CDF of your distribution.</em>

As above, for some distributions the CDF will have a closed form expression, but for others it will require numerical integration.

Task 3 <em>Implement a baseline sampling algorithm via the transformation </em><em>S </em>= <em>F<sub>S</sub></em><sup>−1</sup>(<em>U</em>) <em>where </em><em>U </em>∼ U(0<em>,</em>1)<em>.</em>

This baseline algorithm will be used to compare against the more efficient Ziggurat algorithm. To implement this baseline algorithm requires inverting the CDF. If you cannot find a closed-form expression for this inverse function, you may need to implement the bisection algorithm, which is summarized below. This algorithm can be used for any monotonic function <em>h</em>(<em>x</em>). The idea is, in order to find a value <em>x </em>where <em>h</em>(<em>x</em>) = <em>u</em>, maintain values <em>x</em><sub>min </sub>and <em>x</em><sub>max </sub>where <em>h</em>(<em>x<sub>min</sub></em>) <em>&lt; u </em>and <em>h</em>(<em>x</em><sub>max</sub>) <em>&gt; u</em>. A point <em>x </em>is chosen at the midpoint between <em>x</em><sub>min </sub>and <em>x</em><sub>max</sub>. Based on the value of <em>h</em>(<em>x</em>), either <em>x</em><sub>min </sub>or <em>x</em><sub>max </sub>is updated to halve the difference between them. (This pseudocode assumes that <em>h </em>is increasing; the same algorithm can be used for monotonically decreasing functions with a slight variation.)

Input: <em>u</em>, <em>x</em><sub>min</sub>, <em>x</em><sub>max </sub>where <em>h</em>(<em>x</em><sub>min</sub>) <em>&lt; u </em>and <em>h</em>(<em>x</em><sub>max</sub>) <em>&gt; u</em>

The tolerance parameter tol determines exactly how precise the result is. Setting tol = 10<sup>−12 </sup>or smaller is often a good choice.

Task 4 <em>Generate at least 1000 samples from your baseline sampling algorithm. Keep track of the time it takes to run on your computer. (In Matlab, this can be done with the commands tic and toc.) Plot an estimated PDF from these samples, and make sure it matches the true PDF.</em>

<h1>3          Set up the Ziggurat</h1>

Before generating samples with the Ziggurat algorithm, the values <em>x</em><sub>1</sub><em>,x</em><sub>2</sub><em>,…,x<sub>n </sub></em>must be precomputed, as well as the corresponding values <em>y</em><sub>1</sub><em>,y</em><sub>2</sub><em>,…,y<sub>n</sub></em>, where <em>y<sub>k </sub></em>= <em>g</em>(<em>x<sub>k</sub></em>) for each <em>k</em>. These

Figure 4: Diagram of the <em>n</em>th region of the Ziggurat algorithm.

numbers should be selected so that each of the <em>n </em>regions have exactly the same area. For <em>k &lt; n</em>, the <em>k</em>th region is a rectangle, so its area is

(<em>x</em><em>k</em>+1 − <em>x</em>1)(<em>y</em><em>k </em>− <em>y</em><em>k</em>+1)<em>.</em>

For the <em>n</em>th region, the area is

To set up the Ziggurat, first choose a guess <em>A </em>for the area of each region. Based on this <em>A</em>, find <em>x</em><sub>2 </sub>so that the 1st region has area <em>A</em>. This can be done using the bisection algorithm. Then find <em>x</em><sub>3 </sub>so that the 2nd region has area <em>A</em>. Continue computing <em>x</em><sub>4</sub><em>,…,x<sub>n</sub></em>. Given the value that you get for <em>x<sub>n</sub></em>, compute the resulting area of the <em>n</em>th region. If this area is less than <em>A</em>, then the original guess for <em>A </em>must have been too large; if greater than <em>A</em>, then the original choice of <em>A </em>must have been too small. Again, the bisection algorithm can be used to find the exact value of <em>A</em>.

Task 5 <em>Calculate </em><em>x</em><sub>1</sub><em>,x</em><sub>2</sub><em>,…,x<sub>n </sub>and </em><em>y</em><sub>1</sub><em>,y</em><sub>2</sub><em>,…,y<sub>n </sub>for three values of </em><em>n: </em>4<em>,</em>32<em>,</em>256<em>. Check to make sure </em><em>x</em><sub>1 </sub><em>&lt; x</em><sub>2 </sub><em>&lt; </em>··· <em>&lt; x<sub>n</sub></em><em>. Be sure to include the code that you used. Also list the </em><em>x<sub>k </sub>and </em><em>y<sub>k </sub>numbers for the </em><em>n </em>= 4 <em>case. (Your report does not need to list the numbers for </em><em>n </em>= 32<em>,</em>256<em>.)</em>

<h1>4          Sampling from the <em>n</em>th region</h1>

Since the <em>n</em>th region of the Ziggurat is not a rectangle, it requires a different algorithm to sample from. However, since this region only comes up with probability 1<em>/n</em>, the algorithm does not need to be especially efficient. Figure 4 shows the <em>n</em>th region. It can be separated into two parts: (i) a rectangle extending horizontally from <em>x</em><sub>1 </sub>to <em>x<sub>n </sub></em>and vertically from 0 to <em>y<sub>n</sub></em>, and (ii) an infinitely long tail extending from <em>x<sub>n </sub></em>to ∞, and upper bounded by <em>g</em>(<em>x</em>).

Task 6 <em>If the point </em><em>X,Y is chosen uniformly from this region, compute the probability </em><em>p that it falls into the rectangular part of the </em><em>nth region.</em>

Remember that we do not really need <em>Y </em>, we only need <em>X</em>. When <em>X,Y </em>is in the rectangular part, then <em>X </em>is simply uniform from <em>x</em><sub>1 </sub>to <em>x<sub>n</sub></em>. Thus, we can do the following: with probability <em>p</em>, generate <em>X </em>∼ U(<em>x</em><sub>1</sub><em>,x<sub>n</sub></em>). Otherwise, generate <em>X </em>from the tail distribution.

Generating <em>X </em>from the tail distribution is the last piece of the Ziggurat algorithm. This requires generating a random variable from the PDF

where the constant <em>c</em><sup>0 </sup>is chosen appropriately.

Task 7 <em>Find and implement a method to sample from the tail distribution </em><em>f<sub>T</sub>. One of two methods can be used, depending on your distribution:</em>

<ol>

 <li><em>If </em><em>g</em>(<em>x</em>) <em>has a closed-form integral, then a version of the inverse-CDF transformation can be used, as in your baseline sampling algorithm.</em></li>

 <li><em>If </em><em>g</em>(<em>x</em>) <em>does not have a closed-form integral, you can use another rejection sampling algorithm. Here you should use the method of Homework 4 problem 3: choose a distribution </em><em>f<sub>R </sub>defined on </em><em>x &gt; x<sub>n </sub>that is easy to sample from, and a constant </em><em>M such that </em><em>Mf<sub>R</sub></em>(<em>x</em>) <em>&gt; g</em>(<em>x</em>) <em>for all </em><em>x &gt; x<sub>n</sub>. For the distribution </em><em>f<sub>R</sub>, you may wish to use the exponential distribution restricted to </em><em>x &gt; x<sub>n</sub>; another option is the Pareto distribution, given by PDF</em></li>

</ol>

<em>where </em><em>α &gt; </em>0 <em>is a parameter. The exact choice of distribution </em><em>f<sub>R </sub>is up to you, but be sure to explain your choice. To implement this rejection sampling algorithm, generate </em><em>X </em>∼ <em>f<sub>R</sub></em><em>, and </em><em>Y </em>∼ U(0<em>,Mf<sub>R</sub></em>(<em>X</em>))<em>. Accept if </em><em>Y &lt; g</em>(<em>X</em>)<em>, and repeat if not. Note that this rejection sampling loop should be done inside a single iteration of the outer rejection sampling loop.</em>

<h1>5          Implement and analyze the Ziggurat algorithm</h1>

Task 8 <em>Complete the implementation of the Ziggurat algorithm for </em><em>n </em>= 4<em>, </em><em>n </em>= 32<em>, and </em><em>n </em>= 256<em>.</em>

Task 9 <em>For each of the three </em><em>n values, run your Ziggurat algorithm to generate at least 1,000,000 samples. Make sure you compute the </em><em>x</em><sub>1</sub><em>,…,x<sub>n </sub>and </em><em>y</em><sub>1</sub><em>,…,y<sub>n </sub>constants only once. For each of the </em><em>n values, use these samples to plot an estimated PDF, and compare against the true PDF. Carefully check that these match for your </em><em>n </em>= 4 <em>algorithm, to make sure that your tail algorithm is working correctly.</em>

Task 10 <em>As you generate samples from the three variants of your algorithm, keep track of the following data:</em>

<ol>

 <li><em>How long it takes to run. Compare the time per sample with your baseline algorithm.</em><a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a> <em>Make sure you run your code on the same computer, so that the comparison in meaningful.</em></li>

 <li><em>How often each of the following possible outcomes occurs in the rejection loop:</em>

  <ul>

   <li><em>X is accepted because </em><em>X &lt; x<sub>k</sub>,</em></li>

   <li><em>X &gt; x<sub>k </sub>but </em><em>X is accepted because </em><em>Y &lt; g</em>(<em>X</em>)<em>,</em></li>

   <li><em>Y &gt; g</em>(<em>X</em>) <em>so </em><em>X is rejected,</em></li>

   <li><em>k </em>= <em>n </em><em>and a sample is drawn from the rectangular part of the </em><em>nth region, (e) </em><em>k </em>= <em>n </em><em>and the tail algorithm is run.</em></li>

  </ul></li>

</ol>

Task 11 <em>Can you think of any ways to further improve the performance of your algorithm?</em>

<h1>6          Distribution assignments</h1>

The following table lists the distribution assignments for each student. Recall that the distribution you are simulating is

where <em>c </em>is a constant that you must determine. The table lists the function <em>g</em>(<em>x</em>) and the boundary point <em>x</em><sub>1</sub>.

Student                                                         Distribution name                   <em>g</em>(<em>x</em>)                                            <em>x</em><sub>1</sub>

Arkan Abuyazid                                            Half-Student’s <em>t                         </em>(1+4<em>x</em><sup>2</sup>)<sup>−5<em>/</em>8                                            </sup>0

Curtis Anderson                                            Half-Cauchy

−<em>x</em><sup>3</sup>

Cooper Bertke                                              Half-generalized Gaussian      <em>e                                                 </em>0

0<em>.</em>5

−<em>x</em><sup>4</sup>

Mihir Kotak                                                   Half-generalized Gaussian      <em>e                                                 </em>0

-Gaussian

Yuye Ran                                                      Restricted inverse-gamma        <em>x</em><sup>−2</sup><em>e</em><sup>−1<em>/x                                                       </em></sup>0<em>.</em>5

Samuel Roark Restricted Erlang                                                                                                    <em>x</em><sup>3</sup><em>e</em><sup>−<em>x</em></sup>

0<em>.</em>5

−0<em>.</em>5

Note: Φ is the standard Gaussian CDF, which can be computed in Matlab using the normcdf function

<a href="#_ftnref1" name="_ftn1">[1]</a> Some of you may find that your baseline algorithm actually runs faster than the Ziggurat algorithm. If this is happens even for <em>n </em>= 256, be sure to check your Ziggurat implementation to make sure it is working correctly. However, even if everything is working right, the baseline algorithm may still run faster. This could happen for two reasons. First, for some distributions with closed-form CDFs, computing the inverse CDF <em>F<sub>S</sub></em><sup>−1 </sup>can be quite fast. Second, since Matlab is an interpreted language, it sometimes runs quite slowly compared to the same algorithm implemented in a language like C. For example, Matlab is notoriously slow when it comes to loops. Since the goal of this project is for you to understand the algorithm rather than to build a commercial product, this is nothing to worry about.