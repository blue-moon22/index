---
title: Hidden Markov Models
date: 2017-05-01
---

Hidden Markov models here, there and everywhere! Here’s a short and sweet description of a Hidden Markov model used to generate a short DNA sequence, and accompanying R code.

### Brief rundown of the Hidden Markov Model (HMM)
The nucleotides are basically the letters that make up a DNA sequence: “A”, “T”, “G” and “C”. In a sequence generated by the Markov Model, the probability of finding a nucleotide (or letter) at a particular position in a sequence depends on the nucleotide found at the previous position. For example, if the previous nucleotide is “A”, then the probabilities of finding another “A”, “T”, “G” or “C” could be 0.2, 0.3, 0.4 or 0.1, respectively. Notice how these probabilities add to 1.

The HMM goes a little bit further. The probability of finding a nucleotide at a particular position in a sequence depends on the state rather than the nucleotide of the previous position. What state the position is in also depends on the state of the previous position. The state could be a property of that position in the sequence. For example, a position may belong to either one of two states, “GC-rich” or “AT-rich”. (More complex HMM models may have states such as “promoter”, “exon” and “intron” etc.)

### Generating a sequencing using HMM

Say there are two roulette wheels, one is “GC-rich” and “AT-rich”. Both roulette wheels have four slices labelled “A”, “T”, “G” and “C” but taking up different fractions of the roulette wheel, e.g. “G” and “C” will take up more space on the “GC-rich” wheel than “AT-rich” and vice versa for “A” and “T”.

To generate the first nucleotide of the sequence, I need to choose the first wheel to spin. I flip an unbiased coin to choose one roulette wheel: heads for “GC-rich” and tails for “AT-rich”, each with a probability of 0.5. I flip heads so spin the “GC-rich” roulette wheel. The nucleotide in the sequence will be where the arrow in the wheel stops, either in "A", "T", "G" or "C". For generating the second nucleotide, I need to decide which roulette wheel to use next, which is dependent on the wheel I already spun. For example, there is 90% chance that I will use the “GC-rich” roulette wheel again and a 10% chance I will switch to “AT-rich”. This then repeats again until I get my sequence.

To create the model in R, we need two matrices to hold these probabilities. The first is the transition matrix that contains the probabilities of switching from one state to the other. The rows correspond to the states of the previous position and the columns correspond to the states of the current position. For example, there is a 0.7 chance of staying in state AT-rich and a 0.3 chance of switching to state GC-rich from AT-rich.

#### Transition Matrix

<table>
  <tbody>
    <tr>
      <td></td>
      <td>AT-rich</td>
      <td>GC-rich</td>
    </tr>
    <tr>
      <td>AT-rich</td>
      <td>0.7</td>
      <td>0.3</td>
    </tr>
    <tr>
      <td>GC-rich</td>
      <td>0.1</td>
      <td>0.9</td>
    </tr>
  </tbody>
</table>

The second one is the emission matrix that contains the probabilities of choosing the four nucleotides “A”, “T”, “G” and “C” in each of the states. For example, there is a 0.39 chance of choosing “A” when the current state is AT-rich.

#### Emission Matrix

<table>
  <tbody>
    <tr>
      <td></td>
      <td>A</td>
      <td>C</td>
      <td>G</td>
      <td>T</td>
    </tr>
    <tr>
      <td>AT-rich</td>
      <td>0.39</td>
      <td>0.10</td>
      <td>0.10</td>
      <td>0.41</td>
    </tr>
    <tr>
      <td>GC-rich</td>
      <td>0.10</td>
      <td>0.41</td>
      <td>0.39</td>
      <td>0.10</td>
    </tr>
  </tbody>
</table>

Below is R code that calls a function “generateHMMSeq” and creates a sequence of 30 nucleotides from “thetransitionmatrix” and “theemissionmatrix”. Enjoy!

#### Code

```
# Function to generate a DNA sequence, given a HMM and the length of the sequence to be generated.
generateHMMSeq <- function(transition_matrix, emission_matrix, initial_probs, seq_length,
                           nucleotides = c("A", "C", "G", "T"), states = c("AT-rich", "GC-rich")) {
  # Initialise a vector for storing the new sequence
  new_sequence <- character()
  # Initialise a vector for storing the states of each position in sequence
  new_states <- character()

  # Choose the state for the first position
  first_state <- sample(states, 1, rep=TRUE, prob=initial_probs)
  # Get the probabilities of the current nucleotide, given that we are in the "first_state"
  probabilities <- emission_matrix[first_state,]
  # Choose the nucleotide for the first position in the sequence:
  first_nucleotide <- sample(nucleotides, 1, rep=TRUE, prob=probabilities)
  # Store the first nucleotide in the first position of new_sequence
  new_sequence[1] <- first_nucleotide
  # Store the first state in the first position of new_states
  new_states[1] <- first_state

  for (i in 2:seq_length) {
    # Get the previous state
    prev_state <- new_states[i-1]
    # Get the probabilities of the current state, given that the previous nucleotide was generated by state "prevstate"
    state_probs <- transition_matrix[prev_state,]
    # Choose the state for the ith position in the sequence
    state <- sample(states, 1, rep=TRUE, prob=state_probs)
    # Get the probabilities of the current nucleotide, given that we are in the state "state"
    probabilities <- emission_matrix[state,]
    # Choose the nucleotide for the ith position in the sequence
    nucleotide <- sample(nucleotides, 1, rep=TRUE, prob=probabilities)
    # Store the nucleotide for the current position of the sequence
    new_sequence[i] <- nucleotide
    # Store the state that the current position in the sequence
    new_states[i] <- state
  }

  for (i in 1:length(new_sequence)) {
    nucleotide <- new_sequence[i]
    state <- new_states[i]
    print(paste("Position", i, ", State", state, ", Nucleotide = ", nucleotide))
  }
}

# Define probabilities for AT-rich and GC-rich
states <- c("AT-rich", "GC-rich") # Define the names of the states
ATrichprobs <- c(0.7, 0.3) # Set the probabilities of switching states, where the previous state was "AT-rich"
GCrichprobs <- c(0.1, 0.9) # Set the probabilities of switching states, where the previous state was "GC-rich"
thetransitionmatrix <- matrix(c(ATrichprobs, GCrichprobs), 2, 2, byrow = TRUE) # Create a 2 x 2 matrix
rownames(thetransitionmatrix) <- states
colnames(thetransitionmatrix) <- states

# Define probabilities for A, C, G and T
nucleotides <- c("A", "C", "G", "T") # Define the alphabet of nucleotides
ATrichstateprobs <- c(0.39, 0.1, 0.1, 0.41) # Set the values of the probabilities, for the AT-rich state
GCrichstateprobs <- c(0.1, 0.41, 0.39, 0.1) # Set the values of the probabilities, for the GC-rich state
theemissionmatrix <- matrix(c(ATrichstateprobs, GCrichstateprobs), 2, 4, byrow = TRUE) # Create a 2 x 4 matrix
rownames(theemissionmatrix) <- states
colnames(theemissionmatrix) <- nucleotides

generateHMMSeq(thetransitionmatrix, theemissionmatrix, c(0.5,0.5), 30)
```