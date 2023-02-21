# Constructing a State Space Graph


# Fundamentals of Artificial Intelligence (DSP332) - RTU 2023

Here is the code I used to solve the travellers, bridge and torch problems. I decided to use Python3 because it is easy to write small scripts and also easily readible. It solves the version with 4 travellers who have to all cross a bridge in 15 minutes but the algorithm can easily be adapted for the other versions of this problem.

```python3
from copy import deepcopy
from collections import deque

A = 1
B = 2
C = 5
D = 8

unique_id = 0

class Queue:
    def __init__(self):
        self.elements = deque()

    def enqueue(self, elt):
        self.elements.append(elt)

    def dequeue(self):
        return self.elements.popleft()

    def isempty(self):
        return len(self.elements) == 0

class Node:
    def __init__(self, identifier, time_left, P1 = None, P2 = None, level = None, parents = None):
        self.identifier = identifier
        self.time_left = time_left
        self.P1 = P1 if P1 else []
        self.P2 = P2 if P2 else []
        self.level = level if level else 0
        self.parents = parents if parents else []

    def is_terminated(self):
        maxi = max_list(self.P1)
        return len(self.P1) == 0 or maxi > self.time_left

    def same_parents(self):
        for elt in self.parents:
            if elt[0] == self.P1 and elt[1] == self.P2:
                return True
        return False

    def is_equivalent_on_level(self, graph):
        for node in graph.nodes:
            if node.level == self.level and len(self.P1) == len(node.P1):
                boolean = True
                i = 0
                while i < len(node.P1) and boolean:
                    if node.P1[i] not in self.P1:
                        boolean = False
                    i += 1
                if boolean:
                    return True
        return False

    def display_node(self):
        print(str(self.level) + " | ", end = '')
        print(self.P1, end = '')
        print(" | ", end = '')
        print(self.P2, end = '')
        print(" | " + "time left: " + str(self.time_left))

class Graph:
    def __init__(self):
        self.nodes = []
        self.arcs = []

    def add_node(self, node):
        self.nodes.append(node)

    def add_arc(self, node0, node1):
        self.arcs.append((node0.identifier, node1.identifier))

    def display_graph(self, id_start, nb_tabs):
        for elt in self.arcs:
            if elt[0] == id_start:
                print(nb_tabs * "  ", end = '')
                for node in self.nodes:
                    if node.identifier == elt[1]:
                        node.display_node()
                        self.display_graph(node.identifier, nb_tabs + 1)

    # type_move:
    # 0: from P1 to P2
    # 1: from P2 to P1

    def create_node(self, type_move, node, elt1, elt2 = None):
        global unique_id
        new_node = None
        if elt2 != None:
            new_node = Node(unique_id + 1, node.time_left - max(elt1, elt2), deepcopy(node.P1), deepcopy(node.P2), node.level + 1, deepcopy(node.parents))
            if type_move:
                new_node.P1.remove(elt1)
                new_node.P1.remove(elt2)
                new_node.P2.append(elt1)
                new_node.P2.append(elt2)
            else:
                new_node.P2.remove(elt1)
                new_node.P2.remove(elt2)
                new_node.P1.append(elt1)
                new_node.P1.append(elt2)
        else:
            new_node = Node(unique_id + 1, node.time_left - elt1, deepcopy(node.P1), deepcopy(node.P2), node.level + 1, deepcopy(node.parents))
            if type_move:
                new_node.P1.remove(elt1)
                new_node.P2.append(elt1)
            else:
                new_node.P2.remove(elt1)
                new_node.P1.append(elt1)

        if not(new_node.same_parents()) and new_node.time_left >= 0 and len(new_node.P2) != 0 and not(new_node.is_equivalent_on_level(self)):
            new_node.parents.append((node.P1, node.P2))
            self.add_node(new_node)
            self.add_arc(node, new_node)

            unique_id += 1

            return new_node

        return None

    def create_level(self, node, queue):
        if not(node.level % 2):
            for i in range(len(node.P1)):
                if len(node.P1) == 1:
                    new_node = self.create_node((node.level + 1) % 2, node, node.P1[0])
                    if new_node and not(new_node.is_terminated()):
                        queue.enqueue(new_node)
                for j in range(i + 1, len(node.P1)):
                    new_node = self.create_node((node.level + 1) % 2, node, node.P1[i], node.P1[j])
                    if new_node and not(new_node.is_terminated()):
                        queue.enqueue(new_node)
        else:
            for i in range(len(node.P2)):
                new_node = self.create_node((node.level + 1) % 2, node, node.P2[i])
                if new_node and not(new_node.is_terminated()):
                    queue.enqueue(new_node)
                for j in range(i + 1, len(node.P2)):
                    new_node = self.create_node((node.level + 1) % 2, node, node.P2[i], node.P2[j])
                    if new_node and not(new_node.is_terminated()):
                        queue.enqueue(new_node)

def max_list(array):
    if len(array) == 0:
        return None
    maxi = array[0]
    for elt in array:
        if elt > maxi:
            maxi = elt
    return maxi

def main():
    graph = Graph()
    queue = Queue()

    first_node = Node(0, 15, [A, B, C, D], [], 0)
    graph.add_node(first_node)
    queue.enqueue(first_node)

    while not(queue.isempty()):
        e = queue.dequeue()
        graph.create_level(e, queue)

    print("Program made by Paul FOURNILLON. (https://kaahos.github.io/)")
    print("State Space Graph for the travellers, bridge and torch problem in the following configuration:")
    print("4 travellers: A = 1min, B=2min, C=5min, D=8min")
    print("time to travel: 15min \n")
    print("Level of the Node in the tree | Node.P1 | Node.P2 | time left\n")
    graph.display_graph(0, 0)
    print("\ntotal number of nodes: " + str(len(graph.nodes)))
    print("This program can easily be adapted for the other versions of this problem.")
    print("All you have to do is to modify the parameters of the first_node in the main function.")

if __name__ == "__main__":
    main()
```

The following code can also be downloaded on my [GitHub page](https://github.com/kaahos/BridgeTorchProblems).

