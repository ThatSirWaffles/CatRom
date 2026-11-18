> [!IMPORTANT]
> This repository is an archive of ecurtiss aka toaditive's work. None of the code here is mine, nor do I reserve any rights to it. This only exists to keep an archive of the project and its documentation alive.

# CatRom

Creates Catmull-Rom splines for Roblox.

## Introduction

A [Catmull-Rom spline](https://web.archive.org/web/20251203222052/https://en.wikipedia.org/wiki/Centripetal_Catmull%E2%80%93Rom_spline) is a C¹ cubic spline that passes through each of its control points. It is a good choice for your project if you want a smooth curve that:

1. Passes through every control point.
2. Does not need any extra data to sculpt it, unlike Bézier curves.

In addition to the control points, CatRom provides two parameters to adjust the spline:

1. `alpha`: A number, usually in `[0, 1]`, that loosely affects the curvature of the spline at the control points. The default is `0.5`.
2. `tension`: A number, usually in `[0, 1]`, that makes the spline more or less taut. The default is `0`.

### Reparametrization

By construction, a Catmull-Rom spline clusters points in regions of higher curvature. This effect is often visually unappealing, so CatRom offers a second parametrisation, called a **unit-speed** or **arc-length parametrisation**, that yields equally spaced points given equally spaced times.

By default, passing `true` into a method with a `unitSpeed` argument performs a slow but highly accurate reparametrisation.

If you are calling a high volume of methods with `unitSpeed` set to `true`, you should instead call `CatRom:PrecomputeUnitSpeedData` beforehand. After the initial cost of the precomputation step, this makes reparametrisations significantly faster at a small cost to accuracy.

## Installation

CatRom is available on [Wally](https://wally.run/package/ecurtiss/catrom):

```toml
catrom = "ecurtiss/catrom@0.6.0"
```

It is also available as a `.rbxm` from the [Releases](https://github.com/ThatSirWaffles/CatRom/releases) page.

## Getting started

Here is an annotated example to get you started. Thorough documentation can be found in the API reference.

### Creating a spline

```lua
local CatRom = require(path.to.CatRom)

local points = {
	Vector3.new(0, 0, 0),
	Vector3.new(10, 5, 0),
	Vector3.new(20, 0, 10),
	Vector3.new(30, 10, 20),
}

local alpha = 0.5
local tension = 0

local spline = CatRom.new(points, alpha, tension)
```

The control points can be `Vector2`, `Vector3`, or `CFrame` values.

### Evaluating the spline

Get the position at a single time:

```lua
local pos = spline:SolvePosition(0.5)
```

The time parameter ranges from `0` to `1`.

### Bulk evaluation

Get the position at 100 evenly spaced times between `0` and `1`, inclusive:

```lua
spline:SolveBulk(function(segment, t)
	local pos = segment:SolvePosition(t)
end, 100, 0, 1)
```

### Unit-speed parametrisation

For repeated unit-speed calculations, precompute the required data first:

```lua
spline:PrecomputeUnitSpeedData()
```

Then pass `true` to methods with a `unitSpeed` parameter:

```lua
local pos = spline:SolvePosition(0.5, true)
```

You can also use unit-speed parametrisation with bulk evaluation:

```lua
spline:SolveBulk(function(segment, t)
	local pos = segment:SolvePosition(t)
end, 100, 0, 1, true)
```

### Transporting a CFrame

You can smoothly sweep a `CFrame` from time `0` to `1` with minimal twisting:

```lua
local initialCF = CFrame.new(spline:SolvePosition(0))

local interpolant = spline:GetTransportInterpolant(
	initialCF,
	0,
	1
)

for i = 0, 100 do
	local sweptCF = interpolant(i / 100)
end
```

## API Reference

The [API reference](https://catrom.woffl.net/api) contains detailed documentation for the `CatRom` methods and the supporting modules used internally by the library.
