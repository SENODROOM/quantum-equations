# quantum-equations

A math library for the Quantum language (`.sa`), covering equations from `3 + 4 * 2` up to nonlinear systems, ODEs and symbolic derivatives. It is pure Quantum, with no native code, and 73 tests cover it.

```python
print(Eq.solve("x^2 - 5x + 6 = 0")["roots"])                    # [2, 3]
print(Eq.solve("cos(x) = x")["roots"])                          # [0.7390851332151607]
print(Eq.solveSystem(["x + y = 3", "x - y = 1"], ["x", "y"]))   # unique: x = 2, y = 1
print(Eq.derivative("x^2 * sin(x)", "x"))                       # 2x*sin(x) + x^2*cos(x)
print(Eq.expand("(x + 1)^3"))                                   # x^3 + 3x^2 + 3x + 1
```

## Using it

Quantum has no runtime module loader yet, so the library is combined ahead of time with
[quantum-bundle](../../QuantumLogicsLabs/QuantumLanguage/compiler/qpmexamples/quantum-bundle). There are two ways in:

1. **Single file.** Copy [dist/quantum-equations.sa](dist/quantum-equations.sa) next to your program, or concatenate it in front of your code. It defines all the globals below.
2. **Bundler.** Put `# @use path/to/quantum-equations/index.sa` at the top of your file and bundle it. Each module can also be used alone, for example `# @use .../src/poly.sa`.

```bat
:: from this folder (see package.json "scripts")
qrun C:/QuantumLogicsLabs/QuantumLanguage/compiler/qpmexamples/quantum-bundle/bin/bundle.sa
qrun build/test.sa          :: 73 tests
qrun build/demo.sa          :: guided tour, simple to complex
```

## What's in it

| Global | File | Covers |
|---|---|---|
| `Eq` | [src/eq.sa](src/eq.sa) | **Equations as text**: `solve`, `solveSystem`, `expand`, `taylor`, `table`, `integral`, `limit`, `check`, `equivalent` |
| `Expr` | [src/expr.sa](src/expr.sa) | Formula parser: `evaluate`, `compile`, `simplify`, `derivative` (symbolic), `show`, `variables` |
| `Solve` | [src/solve.sa](src/solve.sa) | `linear`, `quadratic`, `cubic`, `quartic`, `bisection`, `brent`, `newton`, `secant`, `fixedPoint`, `findRoots`, `minimize`, `newtonSystem`, `gaussSeidel` |
| `Poly` | [src/poly.sa](src/poly.sa) | Polynomials as coefficient arrays: arithmetic, division, `roots`, `distinctRoots`, `fromRoots`, `interpolate`, `fit` |
| `Calc` | [src/calculus.sa](src/calculus.sa) | `derivative`, `gradient`, `jacobian`, `hessian`, `limit`, `integrate` (incl. infinite ranges), `simpson`, `romberg`, `gauss`, `integrate2`, `arcLength`, ODEs `euler` / `rk4` / `solveIVP` (scalar or system) |
| `LinAlg` | [src/linalg.sa](src/linalg.sa) | `solve`, `det`, `inverse`, `rank`, `rref`, `analyze` (unique / none / infinite), `leastSquares`, `eigenvalues`, `eigenSymmetric`, `powerIteration`, `charPoly` |
| `Complex` | [src/complex.sa](src/complex.sa) | Complex arithmetic, `sqrt`, `pow`, `roots`, `exp`, `ln`, `show` |
| `Frac` | [src/fraction.sa](src/fraction.sa) | Exact rationals, exact `solveSystem` and `det`, `fromNumber` |
| `NumTheory` | [src/numtheory.sa](src/numtheory.sa) | `linearDiophantine`, `linearCongruence`, `crt`, `pell`, `modPow`, `modInverse`, `isPrime`, `factorize` |
| `Num` | [src/core.sa](src/core.sa) | `gamma`, `erf`, `sinh` & co., `cbrt`, `fmt`, `approx`, `binomial`, `linspace` |

## Solving equations from text

```python
Eq.solve("3(x - 2) = x + 4")                 # linear
Eq.solve("x^2 + 2x + 5 = 0")                 # roots [], complex [-1 - 2i, -1 + 2i]
Eq.solve("(x - 2)^3 (x + 1) = 0")            # roots [-1, 2], multiplicities [1, 3]
Eq.solve("a x + b = 0", "x", {"a": 2, "b": -6})
Eq.solve("sin(x) = 0.5", "x", {}, {"range": [0, 7]})   # numeric scan -> 3 roots
Eq.solve("x^3 + sin(x) = 2", "x", {}, {"guess": 1})    # Newton from a start
```

`solve` returns `{"variable", "method", "roots", "multiplicities", "complex", "degree", "polynomial", "identity"}`.

- **Polynomial** equations of any degree (including rational forms like `(x^2-4)/2 = 0` and parameters supplied in `vars`) are solved exactly, with multiplicities and complex roots.
- **Everything else** is solved numerically. `options.range` (default `[-100, 100]`) is scanned for sign changes and each bracket is refined with Brent's method. Poles such as `1/x` are not reported as roots. Roots where `f` only touches zero (like `sin(x)^2 = 0`) can be missed. Pass `"guess"` for those.

### Systems

```python
Eq.solveSystem(["2x + 3y - z = 5", "x - y + 2z = 1", "3x + y + z = 8"], ["x", "y", "z"])
# {"type": "unique", "solution": {"x": 4.2, "y": -2, "z": -2.6}}
# "type" can also be "none", or "infinite" with a "particular" solution and the "free" unknowns.

Eq.solveSystem(["x^2 + y^2 = 25", "y = x + 1"], ["x", "y"])
# {"type": "nonlinear", "solutions": [{"x": 3, "y": 4}, {"x": -4, "y": -3}]}
```

Linear systems are detected automatically. Nonlinear systems use damped Newton from several starting points, or from `options.guess`.

## Formula language

`+ - * / % ^` (also `**`), `!`, parentheses, implicit multiplication (`2x`, `3(x+1)`, `(x+1)(x-1)`, `2pi`), constants `pi e tau phi inf`, and these functions:
`sin cos tan sec csc cot asin acos atan atan2 sinh cosh tanh asinh acosh atanh sqrt cbrt root abs sign exp ln log log2 log10 floor ceil round trunc min max pow mod hypot fact gamma erf deg rad`.

- `-2^2` is `-4`, and `2^3^2` is `512`.
- `log(x)` is base 10, `ln(x)` is natural, and `log(x, b)` is base `b`.
- Division by zero gives `±Infinity` or `NaN` instead of raising, so root scans and integrals survive poles.
- `Expr.compile("x^2 + a", "x", {"a": 1})` returns an ordinary function. Use it with `Calc`, `Solve`, or your own code.

## Numeric notes

- Doubles throughout: about 15 digits. `Num.fmt(x, digits)` prints more than the native `str()`, which keeps only 10 significant digits.
- Repeated polynomial roots from `Poly.roots` are accurate to about 1e-5. `Poly.distinctRoots` (used by `Eq.solve`) re-solves each cluster and restores full accuracy.
- `NumTheory` and `Frac` are exact up to 2^53 (about 9.0e15) and raise or return `null` beyond that.

## Quantum VM rules this library follows

See the package README in `qpmexamples` for the general list. Three more were found while building this library:

- **Inside a class, a bare call like `abs(x)` resolves to the class's own method of that name** if it has one. Classes here that define `abs`, `sqrt`, `pow`, `sin`, `cos` or `num` call the natives as `Math.abs(...)` and so on instead.
- **Identifiers starting with `split`** after a dot are rewritten by the front-end, so there is no `splitEquation` (it is `sidesOf`).
- **`hypot` and `cbrt` exist only on `Math`**, not as globals. Division by zero raises natively, so the library guards its own divisions.

## Layout

```
index.sa                bundle entry (lists the modules in dependency order)
src/                    the modules
test/*.test.sa          suites, run together by test/all.test.sa
examples/demo.sa        guided tour
dist/quantum-equations.sa   pre-bundled single file (rebuild with the bundle command above)
```
