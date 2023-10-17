---
layout: post
title: Crafting Fairness, Board Game League Scheduling with Local Search
subtitle: by Lucas Zhang
cover-img: /assets/img/Game Schedule.png
thumbnail-img: /assets/img/Game Schedule.png
share-img: /assets/img/Game Schedule.png
tags: [Artificial Intelligence]
---

Scheduling weekly games for a board game league can be a daunting task, especially when fairness is a top priority. 

### The Scheduling Challenge

In this blog post, we will dive into the intriguing world of local search algorithms to create a balanced and equitable league schedule for our 13-player board game league. To keep it consistent, we've named our players A, B, C, D, E, F, G, H, I, J, K, L, and M.

Our challenge is to design a 13-week league schedule where each week, one player receives a "bye" (a week off), and the other 12 players participate in three games, with each player playing one game. The aim is to create a schedule that meets the following fairness criteria:

*  Player Versatility: Each player should play against every other player at least once throughout the league.

*  Diverse Opponents: Each player should have different opponents as often as possible to promote variety and fairness.

*  No Repeat Games: Avoid having the exact same four players play against each other in multiple weeks if possible.

### Setting up the Local Search

We'll start by representing a full 13-week league schedule using a complete-state formulation. Then, we'll construct an objective function that scores the schedule based on the fairness metrics mentioned above. This objective function will guide the local search algorithms in finding an optimal league schedule.

{% highlight python %}
import random
import numpy as np
import copy
import math

class Node:
    def __init__(self, teams):
        self.teams = teams
        self.n_teams = 13
        self.n_weeks = 13
        
        self.schedule = []
        for i in range(self.n_weeks):
            teams_r = random.sample(self.teams, self.n_teams)
            week = [set() for _ in range(4)]
            week[0] = {teams_r[0]}
            week[1] = set(teams_r[1:5])
            week[2] = set(teams_r[5:9])
            week[3] = set(teams_r[9:13])
            self.schedule.append(week)
        
        # array to record how many times each team encounter the other
        # excluding itself (n_teams-1) 
        self.team_matchups = np.zeros((self.n_teams-1, self.n_teams-1), dtype=int)
        self.summary()
        
        self.fairness_score = 10000
        self.distinct_matchs = set(frozenset(week[i]) for week in self.schedule for i in range(1,4))
        self.evaluation()
            
    def summary(self):
        team_matchups = np.zeros((self.n_teams, self.n_teams), dtype=int)
        for t1 in range(self.n_teams):
            for t2 in range(self.n_teams):
                if t1 != t2: # the diagonal should be initialized to 0
                    for week in self.schedule:
                        for match in week:
                            if self.teams[t1] in match and self.teams[t2] in match:
                                team_matchups[t1][t2] += 1
        self.team_matchups = team_matchups[~np.eye(self.n_teams, dtype=bool)].reshape(self.n_teams, self.n_teams - 1)
        # remove the diagonal elements
        
    def evaluation(self):
        self.fairness_score = 10000
        # a player hasn't played with another
        num_zeros = np.count_nonzero(self.team_matchups == 0)
        self.fairness_score -= num_zeros*100
        # having more distinct matches to avoid same 4 player
        self.fairness_score += len(self.distinct_matchs)  # maximum number = 39
        # play different opponents as much as possible
        row_sd_sum = np.sum(np.std(self.team_matchups, axis=1))
        self.fairness_score -= row_sd_sum # smaller std, fairer
        
    def print(self):
        for week in self.schedule:
            player = week[0]
            matches = week[1:4]
            print(f"Bye for Player {player}: {matches}")
        print('** Replay Summary **')
        for t1 in range(self.n_teams):
            print(f"{self.teams[t1]}: ", end="")
            for t2 in range(self.n_teams):
                if t1 != t2:
                    if t1 > t2:
                        print(f"{self.teams[t2]}: {self.team_matchups[t1][t2]}, ", end="")
                    if t1 < t2:
                        print(f"{self.teams[t2]}: {self.team_matchups[t1][t2-1]}, ", end="")
            print()  # Print a new line after each row of matchups
        print(f"Number of matches with identical four players: {39 - len(self.distinct_matchs)}")
        print(f"fairness_score: {self.fairness_score}")
        
    def random_childnode(self):
        child = copy.deepcopy(self)
        
        random_week = random.randint(0, child.n_weeks - 1)
        random_match = random.sample(range(4), 2)
        matchA = list(child.schedule[random_week][random_match[0]])
        matchB = list(child.schedule[random_week][random_match[1]])
        playerA = matchA.pop(random.sample(range(len(matchA)), 1)[0])
        playerB = matchB.pop(random.sample(range(len(matchB)), 1)[0])
        matchB.append(playerA)
        matchA.append(playerB)
        
        child.schedule[random_week][random_match[0]] = set(matchA)
        child.schedule[random_week][random_match[1]] = set(matchB)
        child.summary()
        child.evaluation()
        return child
{% endhighlight %}

### Hill Climbing

Hill climbing is a local search algorithm that iteratively makes small changes to the current solution in the hopes of reaching a better one. In the code below, we'll implement a version of First-Choice Hill Climbing with Random Restarts to find a schedule that maximizes fairness.

First-choice hill climbing employs a stochastic hill climbing approach, where it randomly generates successor solutions until one surpasses the quality of the current state. This strategy proves effective when a state boasts a multitude of potential successors. In the code below, we will also restart the program to find the best schedule.

{% highlight python %}
print("Using First-choice hill climbing algorithms with random restarts:")
teams = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M"]
max_restarts = 10
max_child_search = 1000
max_steps = 50
current_best = Node(teams)
for k in range(max_restarts):
    parent = Node(teams)
    for j in range(max_steps):
        flag = False
        for i in range(max_child_search):
            child = parent.random_childnode()
            if child.fairness_score > parent.fairness_score:
                parent = child
                flag = True
        if not flag:
            print("Exit searching, no better solution found")
            break  # No better child found, exit the loop of j
        print("Find a better schedule!")
    if parent.fairness_score > current_best.fairness_score:
        current_best = parent
        print("The schedule found in this search is better! Update current best.")
    print("Restart...")
current_best.print()
{% endhighlight %}


### Simulated Annealing

Simulated annealing is another local search algorithm inspired by the annealing process in metallurgy. It allows us to escape local optima by accepting less favorable moves with a decreasing probability. We'll also implement a version of simulated annealing to search for an improved league schedule.

{% highlight python %}
print("Using simulated annealing algorithms with random restarts:")
teams = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M"]
max_restarts = 3
max_child_search = 1000
max_steps = 50
initial_temperature = 100
cooling_rate = 0.6

current_best = Node(teams)
for k in range(max_restarts):
    parent = Node(teams)
    temperature = initial_temperature
    for j in range(max_steps):
        flag = False
        for i in range(max_child_search):
            child = parent.random_childnode()
            delta_fairness = child.fairness_score - parent.fairness_score
            if delta_fairness > 0 or random.random() < math.exp(delta_fairness / temperature):
                parent = child
                flag = True
        if not flag:
            print("Exit searching, no better solution found")
            break
        print("Update the schedule. Current fairness score:", parent.fairness_score, "temperature:", temperature)
        temperature *= cooling_rate # cooling
    if parent.fairness_score > current_best.fairness_score:
        current_best = parent
        print("The schedule found in this search is better! Update current best.")
    print("Restart...")
current_best.print()
{% endhighlight %}

### Output

Using the algorithms above, we can find a quite balanced schedule for this board game.

** Replay Summary **
A: B: 3, C: 3, D: 3, E: 3, F: 3, G: 3, H: 4, I: 2, J: 3, K: 3, L: 3, M: 3, 
B: A: 3, C: 3, D: 3, E: 3, F: 3, G: 2, H: 2, I: 3, J: 3, K: 2, L: 3, M: 3, 
C: A: 3, B: 3, D: 3, E: 3, F: 3, G: 3, H: 4, I: 3, J: 3, K: 4, L: 3, M: 4, 
D: A: 3, B: 3, C: 3, E: 3, F: 3, G: 3, H: 3, I: 3, J: 3, K: 3, L: 3, M: 3, 
E: A: 3, B: 3, C: 3, D: 3, F: 3, G: 3, H: 3, I: 3, J: 3, K: 3, L: 3, M: 3, 
F: A: 3, B: 3, C: 3, D: 3, E: 3, G: 3, H: 3, I: 3, J: 3, K: 3, L: 3, M: 3, 
G: A: 3, B: 2, C: 3, D: 3, E: 3, F: 3, H: 4, I: 3, J: 3, K: 3, L: 3, M: 3, 
H: A: 4, B: 2, C: 4, D: 3, E: 3, F: 3, G: 4, I: 3, J: 3, K: 3, L: 4, M: 3, 
I: A: 2, B: 3, C: 3, D: 3, E: 3, F: 3, G: 3, H: 3, J: 4, K: 2, L: 3, M: 4, 
J: A: 3, B: 3, C: 3, D: 3, E: 3, F: 3, G: 3, H: 3, I: 4, K: 2, L: 3, M: 3, 
K: A: 3, B: 2, C: 4, D: 3, E: 3, F: 3, G: 3, H: 3, I: 2, J: 2, L: 3, M: 2, 
L: A: 3, B: 3, C: 3, D: 3, E: 3, F: 3, G: 3, H: 4, I: 3, J: 3, K: 3, M: 2, 
M: A: 3, B: 3, C: 4, D: 3, E: 3, F: 3, G: 3, H: 3, I: 4, J: 3, K: 2, L: 2, 
Number of matches with identical four players: 0
