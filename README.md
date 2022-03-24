# wordle-solver
Proposed solution for the game Wordle.

***

## Introduction and theory
We have written a puzzle solver for the popular game Wordle. This system is capable of guessing a given 5-letter word through the knowledge that it gains while playing it. Moreover, we will monitor its results as well as its performance.

Wordle is a word puzzle where the player gets 6 chances of guessing a five letter word. After each try, a letter is marked as green if in the right place, yellow if it is in the word but in the wrong place, and grey if it isn’t in the word. The same letter can appear several times in a word. The goal is to finish the puzzle in as few tries as possible

When proposing a solution for solving the puzzle, we got some inspiration from the youtube video [Solving Wordle using information theory](https://www.youtube.com/watch?v=v68zYyaEmEA) by 3blue1brown. His model chooses a word based on the knowledge gain, also known as entropy, that that word has on the current state of the game. Also, both files containing the words are extracted from its GitHub.

Our approach conceives a word as a combination of letters and the proposed algorithm looks for letters that maximise the knowledge gain. Iteratively, through tree search, we look for the letter that maximises entropy and we select the position of that letter in the word by choosing the most probable case (the position in which that letter has the most number of occurrences). Then, we filter all possible words according to the knowledge gained in the iterative process and start the process again.

In information theory entropy is a measurement of the level of information or uncertainty that a variable's possible outcomes contain. The mathematical definition of entropy is:

<img src="https://latex.codecogs.com/svg.image?H(X)&space;=-\sum_{i=1}^nP(x_i)\cdot&space;logP(x_i)" title="\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" />

Where X is a variable with multiple possible outcomes and x is a possible outcome. 

Entropy is a better choice than standard probability for this system since the main goal is to maximise the information gained by each guess.

***

## Implementation

#### Main Logic Classes
The main components of the solver are a __Wordle simulator__ and a __guesser__. The Wordle simulator focuses almost entirely on simulating the back-end of Wordle, with minimal user-interface. The guesser uses search tree to find and guess words based on the likelihood of a specific letter being at a particular position. 

The __WordleSim__ class uses two dictionaries; one containing all allowed words, and one containing all “winning” words. Allowed words are a dictionary containing around 13000 words that are viable as guesses during a session. Winning words are words that can be picked as the answer, and is a smaller set of about 2000 words.

In order to evaluate and communicate the results of a guess, the Wordle simulator first checks if any letters are in the right position, then if any remaining letters are in the word but at the wrong place, and returns an array of numbers. This array contains 2 on the positions where the guessed letter was correct and in the right place, a 1 if the guessed letter was correct but in the wrong place, and a 0 if the letter was not in the word at all.

The __Guesser__ implements the main logic of the algorithm used to find a word. It consists of a Tree that returns a word according to the state of the game. This class performs a search for letter and position till a proposed word is filled. This is where the LetterNodes and PositionNodes come into play, which allow us to calculate entropies to decide what letters and positions are most suited for the state of the game. The Tree is structured like this: Tree > LetterNodes > PositionNodes.

The nodes of the tree are based on letters, and keep track of its parent and prior completed nodes. The nodes also keep track of the letters position in the word. The node is also where the statistical calculations for probability and entropy are implemented. The probability and thus entropy is based on the likelihood of a specific letter occurring at a specific position.

The Guesser is the main link between the Wordle simulation and the Search Tree and keeps track of possible letters, and possible words. Letters are tracked by keeping a list of all letters in the alphabet, removing any letter from the list that is confirmed not to be in the correct word. Possible words are filtered according to prior information.

Filtering is performed by the class **WordsFilter** which takes a list of words, and then for each word checks the following rules that are updated when the evaluation of the WordleSim is performed:
* Does the word have a specific letter in a specific position? 
* Does the word not contain a specific letter in a specific position?
* Does it contain the right amount of the same letter (e.g. sassy and grass contains a different amount of s and should be filtered differently if we know the amount of s in a word.)

If all the conditions are correct, the word is added to the list of approved words used in the next guess.

#### How the solver works
During runtime, this is the process of the solver:
1. First the Wordle simulator is initiated, defining dictionaries and choosing a word. 
2. Then, the Guesser is initiated and the solving can begin. The guesser calls on the tree to provide a guess.
3. The tree constructs a word by first choosing the letter and position of node A with the highest entropy among the root’s children. Then all words are filtered to match the previous rule and the process is repeated until all positions of the word are filled.
4. The guesser takes the response from the tree and suggests it to the Wordle simulator.
5. The Wordle simulator evaluates the guess and returns a result based on how many letters were correctly placed, in the word but incorrectly placed, or not in the word at all.
6. The result is returned to the Guesser which saves any gained information in the WordsFilter class and updates the list of possible words through filtering according to this information. 
7. Based on the information gained by the guesser and based on the new filtered list of words, a new tree is constructed as in 3. Step 3 to 7 is repeated until the word is guessed correctly.

***

## Results
Using the class WordleSolver, we combine the game Wordle (in class WordleSim) and the guesser (Guesser) so that they can play for a given number of iterations. We have put the solver to run for 1000 games, and these are our results:

| Iteration |  1000 |
| Total time |    340 seconds (5min 40s) |
| Mean number of guesses | 5.325 |
| Win rate (less than 7 tries) | 0.837|


## Future work
From our proposed approach we can say that it is quite fast finding a word as well as finding the appropriate words for the 1000 games. Our proposed tree search and filtering delivers a good performance in terms of speed.

When testing our algorithm, we have found that it lacks exploration and that prevents it from achieving a higher win rate. It is also true that it always finds the correct word for the game, but 15% of the time the algorithm takes some more steps (=loses) than it is allowed.

Further improvement might be achieved by increasing exploration such as prioritising words with five different letters, or words containing more untested letters. Another idea to consider is that the word is made to be solvable by an average English speaker. Weighting words according to how common they are in normal conversations would more closely mimic the actual knowledge base of such a person and give a slimmer dictionary of probable words.
