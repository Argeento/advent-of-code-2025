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
