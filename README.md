# Large Language Model State Machine (llmstatemachine)

# WIP

NOTE! This project is at this point a draft and a technical concept exploring state machine use for driving LLM Agents to success.
 
## Introduction

The llmstatemachine library merges a state machine approach with advanced language 
models like GPT, enhancing their decision-making capabilities. This library is designed to 
steer agents, built using these models, on a defined path. It achieves this by using the 
agent's chat history and custom tools you create using regular Python functions.

This setup means the agent remembers past interactions with the tools (chat history) and uses this 
memory, along with the tools you define, to make informed decisions. 
The prototype runs on the OpenAI chat model and enforces the execution of 
these Python function-based tools.

The core focus of this project is to develop and explore workflows for an agent 
using a state machine structure, where the agent's conversational abilities and
memory (chat history) are central.

## Installation
```bash
pip install llmsstatemachine
```

## Usage
To use the Large Language Model State Machine, follow these steps:

1. Initialize a WorkflowAgentBuilder.
2. Define states and their respective transitions.
3. Build the workflow agent and add a system message to it.
4. Run model step by step until DONE.

## Example: Memory Game Agent

Consider a memory game, where you need to remember and match hidden pairs -
you don't see everything at once. Our library helps a language model play 
such games. It keeps track of what's been revealed and helps the model make 
smart guesses. This showcases how our library can be applied to scenarios 
where you need to make decisions with limited information.

```python
import random
from typing import Tuple

from dotenv import load_dotenv

load_dotenv()

from llmstatemachine import WorkflowAgentBuilder


def initialize_game(num_pairs):
    """Create and shuffle the deck, then display it as a hidden board."""
    init_deck = list(range(1, num_pairs + 1)) * 2
    random.shuffle(init_deck)
    return init_deck, [False] * len(init_deck)


deck, board = initialize_game(10)


def display_board(argument: str) -> Tuple[str, str]:
    """Display current board situation. Hidden cards are marked with X.

    Parameters
    ----------
    argument : str
       Use empty text
    """
    status = " ".join(str(deck[i]) if board[i] else 'X' for i in range(len(deck)))
    return f"display_board: {status}", "INIT"


def flip_card(argument: str) -> Tuple[str, str]:
    """Turn or flip a card at given position.
    Shows the value of that card or hides it.

    Parameters
    ----------
    argument : str
       Position number as text. Positions are from 0 to 9.
    """
    position = int(argument)
    if board[position]:
        board[position] = False
        print(f"< debug not shown to agent {display_board('')[0]} >")
        return f"flip_card: Hide card at position {position}.", "INIT"
    board[position] = True
    print(f"< debug not shown to agent {display_board('')[0]} >")
    next_state = "COMPLETE" if all(board) else "INIT"
    return f"flip_card: Showing card at position {position}. Value is {deck[position]}.", next_state


def game_done(argument: str) -> Tuple[str, str]:
    """Call this to end the game when it has been solved.

        Parameters
        ----------
        argument : str
          Reasoning about game end.
    """
    return argument, "DONE"


builder = WorkflowAgentBuilder()
builder.add_state_and_transitions("INIT", {flip_card, display_board})
builder.add_state_and_transitions("COMPLETE", {game_done})
builder.add_end_state("DONE")

memory_game_agent = builder.build()
memory_game_agent.add_system_message("You are a player of memory game. " +
                                     "In this game you have 10 number pairs in 20 cards. " +
                                     "Cards have been shuffled and they are all face down. " +
                                     "You may flip a card to see the value. " +
                                     "According to the rules of the memory game you can check a pair. " +
                                     "If they are not a pair you must flip them back hidden. " +
                                     "Once you have all pairs found and shown the game is done.")

while memory_game_agent.current_state != "DONE":
    memory_game_agent.step()
print("-= OK =-")
```

## API Reference

### WorkflowAgentBuilder

- `add_state_and_transitions(state_name, transition_functions): Define a state and its transitions.`
- `add_end_state(state_name): Define an end state for the workflow.`
- `build(): Builds and returns a WorkflowAgent.`

### WorkflowAgent

- `trigger(function_call, args): Triggers a transition in the workflow.`
- `add_message(message): Adds a message to the workflow.`
- `step(): Executes a step in the workflow.`

## License
Apache 2.0
