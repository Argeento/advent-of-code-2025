# Advent of Code 2025

My solutions to the [AoC 2025](https://adventofcode.com/2025) challenges written in [Civet](https://civet.dev).

## Day 1: Secret Entrance ⭐⭐

```ts
dial .= 50
dialCrossingZero .= 0
dialOnZero .= 0

for rotation of toNumbers input.replaceAll 'L', '-'
  dialCrossingZero += countHundreds dial + sign(rotation), dial + rotation
  dial += rotation
  dialOnZero += 1 if dial % 100 is 0

function countHundreds(a: number, b: number)
  [min, max] := [a, b].sort asc
  floor(max / 100) - floor((min - 1) / 100)

log dialOnZero, dialCrossingZero
```

## Day 2: Gift Shop ⭐⭐

```ts
ranges := toNumbers input |> toChunks 2

function sumInvalidIds(pattern: RegExp)
  sum for [start, end] of ranges
    sum for id of [start..-end]
      id if pattern.test id.toString()

log sumInvalidIds /^(.*)\1$/
log sumInvalidIds /^(.*)\1+$/
```

## Day 3: Lobby ⭐⭐

```ts
banks := getLines input

function findMax(bank: string, turnOn: number, best = '')
  return int best unless turnOn
  bestIndex := indexOf bank, max bank[..-turnOn]
  findMax bank[bestIndex<..], turnOn - 1, best + bank[bestIndex]

log sum banks.map findMax ., 2
log sum banks.map findMax ., 12
```

## Day 4: Printing Department ⭐⭐

```ts
grid .= getArray2d input
rounds: number[] := []

until rounds.-1 is 0
  compact flatMap loop2d grid, (y, x, c) =>
    {y, x} if c is '@' and adjacentPoints(grid, x, y).filter(.item is '@')# < 4
  |> .map delete grid[&.y][&.x] |> .# |> rounds.push

log rounds.0, sum rounds
```

## Day 5: Cafeteria ⭐⭐

```ts
[r, ingredients] := input.split('\n\n').map toNumbers
ranges := r.map abs |> toChunks 2 |> .sort (a, b) => a.0 - b.0

log sum for ingredient of ingredients
  ranges.some [start, end] => inRange ingredient, start, end

tmp .= ranges.0.0
log sum for [start, end] of ranges
  realStart := max [tmp, start]
  continue if end < realStart
  tmp = end + 1
  end - realStart + 1
```

## Day 6: Trash Compactor ⭐⭐

```ts
[...a, b] := input.trim().split '\n'
lines := a.map toNumbers
signs := b.match(/\S/g)!
rotated := rotateMatrixLeft getArray2d input

log sum for sign, i of signs
  eval lines.map(&[i]).join sign

log sum for c of rotated.map(&.join '').join('').split '     '
  eval toNumbers(c).join c.-1
```

## Day 7: Laboratories ⭐⭐

```ts
grid := getArray2d input
beams .= grid.0.map & is 'S' ? 1 : 0
splits .= 0

for line of grid
  nextBeams := [...beams]
  for char, x of line when beams[x] and char is '^'
    splits += 1
    nextBeams[x] = 0
    nextBeams[x - 1] += beams[x]
    nextBeams[x + 1] += beams[x]
  beams = nextBeams

log splits, sum beams
```

## Day 8: Playground ⭐⭐

```ts
{ UndirectedGraph } from graphology
{ connectedComponents, countConnectedComponents } from graphology-components

boxes := getLines(input).map(toNumbers).map toKeys 'x', 'y', 'z' 
graph := new UndirectedGraph
graph.addNode i for i of [0...boxes#]

edges := flatMap 
  for boxA, i of boxes
    for boxB, j of boxes[i<..]
      distance: getDistance3d boxA, boxB
      nodeA: i
      nodeB: j + i + 1
|> .sort (a, b) => a.distance - b.distance

for edge of edges[...1000]
  graph.addEdge edge.nodeA, edge.nodeB 
  
log connectedComponents(graph).map(.#).sort(desc)[...3].reduce(multiply)

lastEdge := edges[1000..].find (edge) =>
  graph.addEdge edge.nodeA, edge.nodeB
  countConnectedComponents(graph) is 1

log boxes[lastEdge!nodeA].x * boxes[lastEdge!nodeB].x
```
