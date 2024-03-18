[Reference video from StatQuest](https://www.youtube.com/watch?v=viZrOnJclY0&list=PLsrpjPHm_EOo8LhK8JOAqbqNxy-Rd53sE&index=1&ab_channel=StatQuestwithJoshStarmer)

Turn words into numbers. Words with similar meanings will have similar numbers. For example:

great --> 4.2
awesome --> 3.9


So, in order to create a **Neural Network** to figure out what numbers we should associate with each word, the first thing we do is create inputs for each unique word.

!!! tip "Activation Function"
    Number of **Activation Functions** = Amount of numbers we want to associate with each word

In the following example, we have 2 Activation Functions, the **weights** represents the word:

<image src="../images/activation_function.png" width=600>

However the **weights** are now all random, and we will optimize them with **Backpropagation**. To do the Backpropagation, we need to use 1 word to predict another word. 

E.g.: Use `Troll 2` to predict `is`: 
Current word is `Troll 2` therefore the `1`, other words all receive `0`, hence **1000**. And we expect the next word to be `is`, a.k.a **0100**.

<image src="../images/backpropagation.png" width=600>

To make the predictions:

1. we connect the **Activation Functions** to the outputs
2. add **weights** to the connections with random initialization values
3. Run the outputs through **SoftMax** function: because we have multiple outputs for classification
4. We use Cross Entropy loss function for **Backpropagation**


Now looking at the initial state, we can plot the weights of the **Activation Functions(AF)** by putting the first AF on the X-axis, and second AF on the Y-axis. The diagram looks like this: 

<image src="../images/af_weights_plotting.png" width=600>

Since `Troll 2` and `Gymkata` are both movies and appear in the same position in the sentence, we expect **Backpropagation** will make them close on the diagram like this:

<image src="../images/af_weights_plotting_2.png" width=600>

We call the "backpropagated" weight - **Embeddings**


!!! note "Summary"
    We ask Neural Network to assign numbers to the words

# Word2Vec
Word2Vec is the process to create Word Embeddings

## 1. Continuous Bag of Words
This increases the context by using the surrounding words to predict what occurs in the middle.

Use `Troll 2` and `great` to predict `is`

## 2. Skip Gram
This increases the context by using the word in the middle to predict the surrounding words.

Use `is` to predict `Troll 2` and `great`


!!! note "Convention"
    We usually use 100 or more AF to create loooots of **Embeddings** per word.

    We use 3 Million words as training data.

    So the total amount of weights in the NN is: 3,000,000*100*2