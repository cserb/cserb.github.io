---
title: "Build your first blockchain from scratch (#3 Finding the longest chain)"
date: 2019-10-31T13:11:56+01:00
type: "post"
draft: false
---
In the previous articles ([Probabilistic Finality](https://dev.to/cerbivore/build-your-first-blockchain-from-scratch-463p), [DAG](https://dev.to/cerbivore/build-your-first-blockchain-from-scratch-2-dag-4f8k)) we looked into how Probabilistic Finality works and how DAGs can help to find the longest chain.

Today we are going have a look at implementing a basic DAG in [Crystal](https://crystal-lang.org). We will focus on implementing a `DAG::Vertex` class and finding the *vertex* with the longest distance from a given starting vertex.
Each vertex will have a unique name and a [Hash](https://crystal-lang.org/api/0.31.1/Hash.html) to represent its edges pointing to its children.

Here is the complete finished implementation:

```ruby
module DAG
  class Vertex
    alias Name = String

    getter name : Name
    getter edges : Hash(Name, Vertex)

    def initialize(@name, @edges = {} of Name => Vertex)
    end

    def add(edge_to vertex : Vertex) : Void
      @edges[vertex.name] = vertex
    end

    def children : Array(Vertex)
      @edges.values
    end
  end

  extend self

  def distances(
    from vertex : DAG::Vertex,
    visited = Hash(DAG::Vertex::Name, Bool).new(false),
    stack = Hash(DAG::Vertex, Int32).new(0)
  ) : Hash(DAG::Vertex, Int32)

    visited[vertex.name] = true

    sorted_children = vertex.children.sort_by { |c| c.name }
    sorted_children.each do |child|
      if !visited[child.name]
        stack[child] = stack[vertex] + 1
        distances(from: child, visited: visited, stack: stack)
      end
    end

    stack
  end

  def tip_of_longest_branch(from vertex : DAG::Vertex) : Array
    distances = self.distances(from: vertex)

    # Tip of longest branch (chain)
    distances.map { |k, v| [k, v] }.sort_by { |d| d[1].as(Int32) }.last
  end
end
```

Let's break the code apart to understand how it works.

**1. DAG::Vertex Class**

```ruby
module DAG
  class Vertex
    alias Name = String

    getter name : Name
    getter edges : Hash(Name, Vertex)

    def initialize(@name, @edges = {} of Name => Vertex)
    end

    def add(edge_to vertex : Vertex) : Void
      @edges[vertex.name] = vertex
    end

    def children : Array(Vertex)
      @edges.values
    end
  end

  ...

end
```

This should be straight forward. We pass along the `Name` when we initialize a new Vertex. Afterwards we create its edges:

```ruby
v1 = DAG::Vertex.new("A")
v2 = DAG::Vertex.new("B")

v1.add edge_to: v2

pp v1.name #=> "A"
pp v1.children #=> [#<DAG::Vertex:0x7f0f4866ee60 @edges={}, @name="B">]
# same as
pp v1.edges.values #=> [#<DAG::Vertex:0x7f0f4866ee60 @edges={}, @name="B">]
```

You can play with the code [here](https://play.crystal-lang.org/#/r/7wq4/edit)

**2. Distances and the Tip of the longest branch**

```ruby
module DAG

  ...

  extend self

  def distances(
    from vertex : DAG::Vertex,
    visited = Hash(DAG::Vertex::Name, Bool).new(false),
    stack = Hash(DAG::Vertex, Int32).new(0)
  ) : Hash(DAG::Vertex, Int32)

    visited[vertex.name] = true

    sorted_children = vertex.children.sort_by { |c| c.name }
    sorted_children.each do |child|
      if !visited[child.name]
        stack[child] = stack[vertex] + 1
        distances(from: child, visited: visited, stack: stack)
      end
    end

    stack
  end

  def tip_of_longest_branch(from vertex : DAG::Vertex) : Array
    distances = self.distances(from: vertex)

    # Tip of longest branch (chain)
    distances.map { |k, v| [k, v] }.sort_by { |d| d[1].as(Int32) }.last
  end
end

```

To find out the distances starting from block `B1` to `Bn` we use depth first search (DFS), that explores each branch to the end and than moves on to the next one.

```ruby
def distances(
    from vertex : DAG::Vertex,
    visited = Hash(DAG::Vertex::Name, Bool).new(false),
    stack = Hash(DAG::Vertex, Int32).new(0)
  ) : Hash(DAG::Vertex, Int32)

  ...
end
```
We start by passing a starting vertex to the method as `from vertex` parameter. The method has a `visited` parameter that is an empty hash, which has been initialized with a default value of `false` for each key that is unknown. Same applies to the `stack` parameter with a default value of `0`. The stack contains the actual distance values.

```ruby
    ...

    visited[vertex.name] = true

    sorted_children = vertex.children.sort_by { |c| c.name }
    sorted_children.each do |child|
      if !visited[child.name]
        stack[child] = stack[vertex] + 1
        distances(from: child, visited: visited, stack: stack)
      end
    end

    ...
```
The rest is pretty simple as well. We mark the vertex currently passed as `from vertex` parameter as visited and then we move on to its *unvisited* children. On a step forward we give the child a distance which is `parent distance + 1`.

If you think you need to better understand DFS you should watch [this video](https://www.youtube.com/watch?v=7fujbpJ0LB4).


```ruby
  def tip_of_longest_branch(from vertex : DAG::Vertex) : Array
    distances = self.distances(from: vertex)

    # Tip of longest branch (chain)
    distances.map { |k, v| [k, v] }.sort_by { |d| d[1].as(Int32) }.last
  end
```

To find the tip of the longest chain we just look at the distances and pick the vertex with the highest distance value. If there are multiple vertices with the same distance we don't really care which one we pick. Why this is not important will become apparent later in the series.

Now let's give it a try:
```ruby
v1 = DAG::Vertex.new("A")
v2 = DAG::Vertex.new("B")
v3 = DAG::Vertex.new("C")
v4 = DAG::Vertex.new("D")


v1.add edge_to: v2
v2.add edge_to: v3
v3.add edge_to: v4

pp DAG.distances from: v1
#  {#<DAG::Vertex:0x7fe7f2a75e60 @edges={
#    "C" => #<DAG::Vertex:0x7fe7f2a75e40 @edges={
#      "D" => #<DAG::Vertex:0x7fe7f2a75e20 @edges={}, #@name="D">},
#    @name="C">},
#  @name="B"> => 1,
#  #<DAG::Vertex:0x7fe7f2a75e40 @edges={
#    "D" => #<DAG::Vertex:0x7fe7f2a75e20 @edges={}, @name="D">},
#  @name="C"> => 2,
#  #<DAG::Vertex:0x7fe7f2a75e20 @edges={}, @name="D"> => 3}


pp DAG.tip_of_longest_branch from: v1
# [#<DAG::Vertex:0x7fce18464e20 @edges={}, @name="D">, 3]

```
You can try it out yourself [here](https://play.crystal-lang.org/#/r/7x1b/edit) and create more complex structures resembling real life scenarios.

**3. Conclusion**

We found an easy way to create a DAG and using a simple depth first search we were able to find the tip of the longest chain.

Integrating the code into a working blockchain will require some more work, but we'll get to that later on.

**4. Cocol Project**

The [Cocol Project](https://github.com/cocol-project) is an effort to implement a basic, but fully working, blockchain in Crystal — a "Minimum Viable Blockchain"

The code we talked about in this article is part of the [ProbFin](https://gitbub.com/cocol-project/probfin) shard. Here is the [DAG implementation](https://github.com/cocol-project/probfin/blob/master/src/dag.cr) and the [tests](https://github.com/cocol-project/probfin/blob/master/spec/dag_spec.cr)
