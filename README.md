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

## Day 9: Movie Theater ⭐⭐

```ts
reds := getLines(input).map toNumbers
lines := reds.map (red, i) => [red, reds[i + 1] ?? reds.0] |> .map normalizeLine
p1Max .= -Infinity
p2Max .= -Infinity

for redA, i of reds
  for redB of reds[i<..]
    [x1, x2, y1, y2] := normalizeLine [redA, redB]
    area := (x2 - x1 + 1) * (y2 - y1 + 1)
    p1Max = area if area > p1Max
    p2Max = area if area > p2Max and not lines.some [lx1, lx2, ly1, ly2] =>
      lx1 is lx2
        ? lx1 > x1 and lx1 < x2 and ly2 > y1 and ly1 < y2
        : ly1 > y1 and ly1 < y2 and lx2 > x1 and lx1 < x2

function normalizeLine(line: number[][])
  x1 := min [line.0.0, line.1.0]
  x2 := max [line.0.0, line.1.0]
  y1 := min [line.0.1, line.1.1]
  y2 := max [line.0.1, line.1.1]
  [x1, x2, y1, y2]

log p1Max, p2Max
```

## Day 10: Factory ⭐⭐

```ts
{ solve } from yalps

machines := getLines(input).map (line) =>
  targetLights: line.match(/[.#]+/)!0.split('').map & is '#'
  buttons: line.match(/\(.*?\)/g)!map toNumbers
  requiredJoltages: line.match(/{.*/)!0 |> toNumbers

log sum for { buttons, targetLights } of machines
  minClicks .= Infinity
  combinations := for i of [0...2**buttons#]
    [...i.toString(2).padStart(buttons#, '0')].map & is '1'

  for combination of combinations
    lights := false for _ of targetLights
    for button, i of buttons when combination[i] is true
       lights[nr] = not lights[nr] for nr of button
    if isEqual lights, targetLights
      minClicks = min [minClicks, compact(combination)#]

  minClicks

log sum for { buttons, requiredJoltages } of machines
  constraints: any := {}
  for req, i of requiredJoltages
    constraints.`joltage_${i}` = equal: req

  variables: any := {}
  for button, i of buttons
    variables.`button_${i}` = press: 1
    for nr of button
      variables.`button_${i}`.`joltage_${nr}` = 1

  solve
    direction: 'minimize'
    objective: 'press'
    constraints: constraints
    variables: variables
    integers: keys variables
  |> .result
```

## Day 11: Reactor ⭐⭐

```ts
{ DirectedGraph } from graphology

devices := getLines input.replaceAll ':', ''
  .map(.split ' ').map [name, ...outputs] => { name, outputs }

graph := new DirectedGraph
graph.addNode 'out'

for device of devices
  graph.addNode device.name

for device of devices
  for output of device.outputs
    graph.addEdge device.name, output

function countPaths(start: string, end: string)
  moveData := memoize (node: string): number =>
    if node is end then 1
    else sum graph.mapOutNeighbors node, moveData
  moveData start

log countPaths 'you', 'out'
log (+)
  countPaths('svr', 'fft') * countPaths('fft', 'dac') * countPaths 'dac', 'out'
  countPaths('svr', 'dac') * countPaths('dac', 'fft') * countPaths 'fft', 'out'
```

## Day 12: Christmas Tree Farm ⭐⭐

```ts
getLines input.split('\n\n').6
  .map toNumbers
  .filter [w, h, ...p] => w * h > 7 * sum p
  .# |> log
```
