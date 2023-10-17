---
layout: post
title: Solving 8 Puzzle Using Searching Algorithms
subtitle: by Lucas Zhang
cover-img: /assets/img/8Puzzle.png
thumbnail-img: /assets/img/8Puzzle.png
share-img: /assets/img/8Puzzle.png
tags: [Artificial Intelligence]
---

The 8-Puzzle is a captivating sliding tile game, with the 15-Puzzle being its more common counterpart.

### The 8-Puzzle

This puzzle comprises a 3x3 grid filled with numbered tiles, except for one missing tile, often denoted by an empty space or a period. The ultimate objective is to rearrange the tiles in a specific order while adhering to the rule of sliding only one tile into the empty space at a time.

In our exploration of this puzzle, we will aim to reach the following configuration as our target:

$$\begin{bmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & . \\
\end{bmatrix}$$

In this post, we will explore different approaches to solving the classic 8-Puzzle using uninformed search and informed search. We will also delve into specific search techniques, such as Breadth-First Search, Depth-Limited Search, Iterative Deepening, and Informed Search using Heuristics and A\* Search.

##### State Representation and Problem Setup

In the provided code below, we will define a class called Node which serves as the state representation for the 8-Puzzle problem. 

*  Initialization: The Node class is initialized with a 3x3 NumPy array, representing the current state of the 8-Puzzle. 

*  State Visualization: The vis method is used to display the current state of the 8-Puzzle. 

*  Child Node Generation: The child_node method generates a new child node by simulating a move in a specified direction ("Left," "Right," "Up," or "Down"). It creates a copy of the current state and swaps the position of the empty cell (0) with the adjacent cell in the chosen direction.

*  Available Actions: The avail_actions method determines the available actions (moves) that can be made from the current state. It returns a list of valid actions by considering the current position of the empty cell (0) and excluding actions that would move the empty cell out of bounds.

{% highlight python %}
import numpy as np
import sys
import time

class Node:
    def __init__(self, stt = np.array([[1,2,3], [4,5,6], [7,0,8]], dtype = int)):
                 # parent = None, path_cost = 0, action_history = None,
                 # goal = np.array([[1,2,3], [4,5,6], [7,8,0]], dtype = int)):

        self.stt = stt
        # self.parent = parent
        # self.path_cost = path_cost
        # self.action_history = action_history
        self.zero_position = np.argwhere(self.stt == 0)[0]
    def vis(self):
        """
        show the current 8-puzzle state.
        """
        print(self.stt)
    def child_node(self, action):  
        stt_temp = np.copy(self.stt)
        row = self.zero_position[0]
        col = self.zero_position[1]
        if action == "Left":
            stt_temp[row, col], stt_temp[row, col + 1] = stt_temp[row, col + 1], stt_temp[row, col]
        if action == "Right":
            stt_temp[row, col], stt_temp[row, col - 1] = stt_temp[row, col - 1], stt_temp[row, col]
        if action == "Down":
            stt_temp[row, col], stt_temp[row - 1, col] = stt_temp[row - 1, col], stt_temp[row, col]
        if action == "Up":
            stt_temp[row, col], stt_temp[row + 1, col] = stt_temp[row + 1, col], stt_temp[row, col]
        return(Node(stt = stt_temp))
    def avail_actions(self):
        """
        Returns
        -------
        a list of available actions
        """
        actions = ["Right", "Left", "Up", "Down"]
        if self.zero_position[0] == 0:
            actions.remove("Down")
        if self.zero_position[0] == 2:
            actions.remove("Up")
        if self.zero_position[1] == 0:
            actions.remove("Right")
        if self.zero_position[1] == 2:
            actions.remove("Left")
        return(actions)
{% endhighlight %}
		
##### Uninformed Search: Breadth-First

Uninformed Search, as the name suggests, operates without any prior knowledge of the problem domain. For the 8-Puzzle, this means shuffling the tiles around until you reach the desired state. Let's start with Breadth-First Search, which is a systematic approach that explores all possible paths, level by level. 

{% highlight python %}
def breadth_first_kernel(initial_state, goal_state):
    initial_node = Node(initial_state)
    visited_stt = set()
    stt_history = []
    action_history = []
    node_queue = [(initial_node, [], [])] # node, stt_history, action_history
    
    while node_queue: # for all the nodes in current layer (and the next layer)
        current_node, stt_history, action_history = node_queue.pop(0)
        current_stt = current_node.stt

        if np.array_equal(current_stt, goal_state):
            return stt_history, action_history
        elif current_stt.tobytes() in visited_stt:
            continue
        else:
            visited_stt.add(current_stt.tobytes())
            for action in current_node.avail_actions():
                child = current_node.child_node(action)
                node_queue.append((child, stt_history + [current_stt],
                                   action_history + [action])) # add nodes in the next layer to search

def breadth_first(initial_state = np.array([[1, 2, 0], [8, 4, 3], [7, 6, 5]], 
                                           dtype=int),
                  goal_state = np.array([[1, 2, 3], [8, 0, 4], [7, 6, 5]], 
                                        dtype=int)):
    stt_history, action_history = breadth_first_kernel(initial_state = initial_state, 
                                                       goal_state = goal_state)
    stt_history.append(goal_state)
    print("breadth_first: Solution found! ")
    print("Action history:", action_history)
    print("State history:")
    for stt in stt_history:
        print(stt)
{% endhighlight %}

##### Uninformed Search: Depth-Limited Search and Iterative Deepening

If we want to search for the deepest level first, it becomes a Depth-First Search. To make it more efficient, we can implement Depth-Limited Search, a variation of Depth-First Search with a depth limit. This avoids getting stuck in infinite paths.

Iterative Deepening performs a series of Depth-Limited Searches with increasing depth limits. It combines the benefits of Breadth-First Search and Depth-First Search. This ensures optimal solutions without a huge memory used.

{% highlight python %}
def depth_first(current_stt = np.array([[1,2,3], [4,5,6], [7,0,8]], dtype = int), 
                goal = np.array([[1,2,3], [8,0,4], [7,6,5]], dtype = int),
                max_depth = 5,
                stt_history = [],
                action_history = []):
    current_node = Node(current_stt)
    if any(np.array_equal(current_stt, stt) for stt in stt_history):
        return None
    elif np.array_equal(current_stt, goal):
        return stt_history, action_history
    elif len(stt_history) >= max_depth:
        return None
    else:
        stt_history = stt_history + [current_stt]
        for action in current_node.avail_actions():
            child = current_node.child_node(action)
            current_stt = child.stt
            action_history = action_history + [action]
            result = depth_first(current_stt, goal, max_depth,
                                 stt_history, action_history)
            if result is not None:
                return result
            if result is None:
                action_history.pop()


def iterative_deepening(initial_state = np.array([[1, 2, 0], [8, 4, 3], [7, 6, 5]], 
                                                 dtype=int),
                        goal_state = np.array([[1, 2, 3], [8, 0, 4], [7, 6, 5]], 
                                              dtype=int),
                        max_depth = 10):
    for limit in range(1, max_depth+1):
        initial_stt_history = []
        initial_action_history = []
        result = depth_first(current_stt = initial_state, 
                             goal = goal_state, 
                             stt_history = initial_stt_history, 
                             action_history = initial_action_history,
                             max_depth = limit)
        if result is None:
            print(f"iterative_deepening: No solution found for the depth of {limit}.")
        else:
            stt_history, action_history = result 
            stt_history.append(goal_state)
            print(f"iterative_deepening: Solution found! Current depth: {limit}")
            print("Action history:", action_history)
            print("State history:")
            for stt in stt_history:
                print(stt)
            break
{% endhighlight %}

##### Informed Search: Heuristics and A\* Search

A\* Search is a powerful and widely used algorithm in the field of artificial intelligence and computer science. It is particularly notable for its efficiency and optimality in finding the shortest path or solution in a search space.

A\* Search leverages a heuristic, which provides an informed estimate of the cost from the current state to the goal. By incorporating this heuristic, A\* Search intelligently explores the most promising paths while systematically examining the search space.

In this post, we'll explore the key concepts behind A\* Search and employ A\* Search for solving 8-Puzzle. We will use two types of Heuristics: the Number of Wrong Tiles, and the Manhattan Distance.

Manhattan Distance $= |x_current - x_target| + |y_current - y_target|$, where we calculate the sum of the horizontal and vertical distances it is away from its desired goal position.

*  Heuristics

{% highlight python %}
def num_wrong_tiles(current_stt, goal):
    return np.sum(current_stt.flatten() != goal.flatten())
def manhattan_distance(current_stt, goal):
    nr, nc = current_stt.shape
    total_distance = 0
    for row in range(nr):
        for col in range(nc):
            value = current_stt[row, col]
            goal_row, goal_col = np.where(goal == value)
            total_distance += abs(row - goal_row) + abs(col - goal_col)
    return total_distance
{% endhighlight %}

*  A\* Search
{% highlight python %}
def astar(initial_state = np.array([[1, 2, 0], [8, 4, 3], [7, 6, 5]], 
                                   dtype=int),
          goal_state = np.array([[1, 2, 3], [8, 0, 4], [7, 6, 5]], 
                                dtype=int),
          heuristic_func = num_wrong_tiles):
    initial_node = Node(initial_state)
    visited_stt = set()
    stt_history = []
    action_history = []
    node_queue = [(100, initial_node, [], [])] # evaluation, node, stt_history, action_history
    
    while node_queue: # for all the nodes in current layer (and the next layer)
        node_queue.sort(key=lambda x: x[0])
        _, current_node, stt_history, action_history = node_queue.pop(0)
        current_stt = current_node.stt

        if np.array_equal(current_stt, goal_state):
            break
        elif current_stt.tobytes() in visited_stt:
            continue
        else:
            visited_stt.add(current_stt.tobytes())
            for action in current_node.avail_actions():
                child = current_node.child_node(action)
                evaluation = len(action_history) + 1 + heuristic_func(child.stt, goal_state)
                node_queue.append((evaluation, child, stt_history + [current_stt],
                                   action_history + [action])) # add nodes to search
    stt_history.append(goal_state)
    print(f"astar search with {heuristic_func.__name__}: Solution found! ")
    print("Action history:", action_history)
    print("State history:")
    for stt in stt_history:
        print(stt)
    stt_history.pop()
    return stt_history, action_history
{% endhighlight %}

##### Comparing the Efficiency

A\* Search is typically more efficient than Uninformed Search algorithms, such as Iterative Deepening, while Breadth-First Search is usually the most time-consuming. We will estimate the time they take to solve the 8-Puzzle below.

{% highlight python %}
initial_state = np.array([[1, 2, 0], [8, 4, 3], [7, 6, 5]], dtype=int)
goal_state = np.array([[1, 2, 3], [8, 0, 4], [7, 6, 5]], dtype=int)
    
start_time = time.time()
breadth_first(initial_state, goal_state)
end_time = time.time()
print(f"Time taken by breadth_first: {end_time - start_time:.6f} seconds")
    
start_time = time.time()
iterative_deepening(initial_state, goal_state)
end_time = time.time()
print(f"Time taken by iterative_deepening: {end_time - start_time:.6f} seconds")
    
start_time = time.time()
astar(initial_state, goal_state, heuristic_func = num_wrong_tiles)
end_time = time.time()
print(f"Time taken by astar with num_wrong_tiles: {end_time - start_time:.6f} seconds")
    
start_time = time.time()
astar(initial_state, goal_state, heuristic_func = manhattan_distance)
end_time = time.time()
print(f"Time taken by astar with manhattan_distance: {end_time - start_time:.6f} seconds")
{% endhighlight %}