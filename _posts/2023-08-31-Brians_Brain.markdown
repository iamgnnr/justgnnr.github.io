---
layout: post
title:  "Cellular Automata and Brian's Brain"
date:   2023-08-31
categories: Python Algorithm Cellular Automata
---

![Brian's Brain](/assets/brians_brain.gif)

Cellular automata are computational models used to simulate complex dynamic systems that evolve according to predefined rules. They are used in a wide variety of fields including physics, biology, and computer science for studying emergent behavior. Originating from the minds of mathematicians John von Neumann and Stanislaw Ulam, these models operate on a grid of cells, each with a finite set of states, evolving over discrete time steps according to pre-defined transition rules. Among the most iconic instances is [John Conway's Game of Life](https://playgameoflife.com/) created in 1970, which still has a thriving community of “players” constantly finding new patterns and emergent phenomena. Cellular automata showcase the remarkable ability of simple rules to create complex patterns and behaviors on a grander scale. Implementations of Conway’s Game of Life are everywhere. For the sake of variety, below I've included the Python code for [Brian’s Brain](https://en.wikipedia.org/wiki/Brian%27s_Brain#:~:text=Brian's%20Brain%20is%20a%20cellular,the%20dying%20cells%20are%20blue.), a lesser known cellular automaton created by [Brian Silverman](https://en.wikipedia.org/wiki/Brian_Silverman):


```
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation

GRID_SIZE = (100, 100)

grid = np.random.choice([0, 1], size=GRID_SIZE)

def update(frameNum, img, grid):
    new_grid = grid.copy()
    for i in range(GRID_SIZE[0]):
        for j in range(GRID_SIZE[1]):
            # Count the number of "on" neighbors
            on_neighbors = np.sum(grid[max(0, i-1):min(GRID_SIZE[0], i+2),
                                       max(0, j-1):min(GRID_SIZE[1], j+2)] == 1) - grid[i, j]
            
            if grid[i, j] == 0:
                if on_neighbors == 2:
                    new_grid[i, j] = 1
            elif grid[i, j] == 1:
                new_grid[i, j] = 2
            elif grid[i, j] == 2:
                new_grid[i, j] = 0
    
    img.set_data(new_grid)
    grid[:] = new_grid[:]
    return img

fig, ax = plt.subplots()
img = ax.imshow(grid, interpolation='nearest', cmap='gray')
ani = animation.FuncAnimation(fig, update, fargs=(img, grid), frames=10, interval=200, save_count=50)

plt.show()


```
