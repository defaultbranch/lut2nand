# lut2nand


## Challenge

For a given LUT, determine the corresponding NAND circuit/formula with the smallest number of gates and/or the shortest gate delay.

Concrete examples could be:

- an _x_-bit adder (find something better than a [ripple-carry-adder](https://en.wikipedia.org/wiki/Adder_(electronics)))
- any _x_-bit integer arithmetic operation


## Considerations

### LUT Representation

The LUT that computes a boolean output for a number of boolean inputs is represented by a boolean array with a lenght of a power of two.

Examples:

- zero inputs:
  - constant 0: `[ 0 ]`
  - constant 1: `[ 1 ]`
- one input:
  - not: `[ 1, 0 ]`
- two inputs:
  - and: `[ 0, 0, 0, 1 ]`
  - or: `[ 0, 1, 1, 1 ]`
  - xor: `[ 0, 1, 1, 0 ]`
  - nand: `[ 1, 1, 1, 0 ]`

### NAND Representation

The  NAND is a term `¬(a ∧ b)` (of two inputs a and b) or a negated for-all quantifier `¬∀` (for an arbitrary set of inputs).

Note: NAND of the empty set `¬∀∅ = false`, from `∀` reading as "all" (true if all qualify, none fails) and therefore `∀∅ = true` (in an empty set, there is no element to fail).

In the [7400-series](https://en.wikipedia.org/wiki/List_of_7400-series_integrated_circuits), NANDs are available with 1 (74x04), 2 (74x00), 3 (74x10), 4 (74x20) or 8 (74x30) inputs.

### LUTs and NANDs

LUTs are general, NANDs are specific logic operators. All LUTS where all except the last value is `1` are NANDs. LUTs are great for abstract considerations (like algorithms or programs), NANDs are great for concrete implementations (like digital circuits, with off-the-shelf logic circuits).

### LUT-to-NAND transformation

#### LUT-to-logic transformation

First, transform the LUT array to a logic expression, `∧` for combining values to rows, `∨` for combining rows to the final result, and `¬` along the way.

##### Example

Considering a one-bit-adder with in/out carry:

|  ci  |  a  |  b  | a+b |  co  |
| ---- | --- | --- | --- | ---- |
|   0  |  0  |  0  |  0  |   0  |
|   1  |  0  |  0  |  1  |   0  |
|   0  |  1  |  0  |  1  |   0  |
|   1  |  1  |  0  |  0  |   1  |
|   0  |  0  |  1  |  1  |   0  |
|   1  |  0  |  1  |  0  |   1  |
|   0  |  1  |  1  |  0  |   1  |
|   1  |  1  |  1  |  1  |   1  |

The three inputs are `ci` (carry-in), `a` and `b`.

The two LUTs are:
  - `a+b` is `[ 0, 1, 1, 0, 1, 0, 0, 1 ]`
  - `co` (carry-out) is `[ 0, 0, 0, 1, 0, 1, 1, 1]`

The logic expressions for the LUTs would then be:

```
a+b = ¬(¬ci ∧ ¬a ∧ ¬b)
    ∨  ( ci ∧ ¬a ∧ ¬b)
    ∨  (¬ci ∧  a ∧ ¬b)
    ∨ ¬( ci ∧  a ∧ ¬b)
    ∨  (¬ci ∧ ¬a ∧  b)
    ∨ ¬( ci ∧ ¬a ∧  b)
    ∨ ¬(¬ci ∧  a ∧  b)
    ∨  ( ci ∧  a ∧  b)
```

and

```
co  = ¬(¬ci ∧ ¬a ∧ ¬b)
    ∨ ¬( ci ∧ ¬a ∧ ¬b)
    ∨ ¬(¬ci ∧  a ∧ ¬b)
    ∨  ( ci ∧  a ∧ ¬b)
    ∨ ¬(¬ci ∧ ¬a ∧  b)
    ∨  ( ci ∧ ¬a ∧  b)
    ∨  (¬ci ∧  a ∧  b)
    ∨  ( ci ∧  a ∧  b)
```

#### Logic-to-NAND Normalization

Next, transform all terms to a normal NAND form.

- `a ∧ b` ⇒ `¬(¬(a ∧ b))`
- `a ∨ b` ⇒ `¬(¬a ∧ ¬b)`

(maybe there are more/better substitution rules)


#### NAND breakdown

Up to this point, there are no intermediate values, only one expression per LUT.

Search and extract subterms (gates), adding one intermediate value per gate, until there are only concrete gates left.

(maybe compute and consider fan-in/fan-out)

Group/order gates by there gate-delay.
