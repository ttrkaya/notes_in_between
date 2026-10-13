# Gaussian falloff, radius expansion, and Marching Cubes

## The fields

Let $x$ be a position in 3D space. Sphere $i$ has center $c_i$ and radius $r_i>0$. The quantity $\lVert x-c_i\rVert$ is the Euclidean distance from $x$ to its center.

The signed distance to that individual sphere is

```math
f_i(x)=\lVert x-c_i\rVert-r_i.
```

The original field takes the minimum over all spheres:

```math
F(x)=\min_i f_i(x)
    =\min_i\bigl(\lVert x-c_i\rVert-r_i\bigr).
```

F is negative where a point is strictly inside at least one ball and positive outside all the balls. Outside the balls, it gives the exact distance to their union. Inside overlapping balls, it generally does not give the signed distance to the combined boundary.

The original reconstruction extracts the zero level:

```math
F(x)=0.
```

The Gaussian field uses one constant positive width $\sigma$, measured in the same length units as F:

```math
G(x)=\exp\left[-\left(\frac{\max(0,F(x))}{\sigma}\right)^2\right].
```

This gives G = 1 wherever F is nonpositive. Outside the balls, G decreases toward zero.

## Deriving the radius expansion

Choose a dimensionless surface level $\tau$ satisfying

```math
0<\tau<1.
```

We want the positions where

```math
G(x)=\tau.
```

Because G is 1 wherever F is nonpositive, this level must have $F(x)>0$. We can therefore replace $\max(0,F(x))$ with $F(x)$:

```math
\exp\left[-\left(\frac{F(x)}{\sigma}\right)^2\right]=\tau.
```

Take the natural logarithm:

```math
-\left(\frac{F(x)}{\sigma}\right)^2=\ln\tau.
```

Rearranging gives

```math
F(x)^2=-\sigma^2\ln\tau.
```

Since F is positive at this level, take the positive square root:

```math
F(x)=\sigma\sqrt{-\ln\tau}.
```

Define the constant expansion distance

```math
\delta=\sigma\sqrt{-\ln\tau}.
```

Substituting the definition of F gives

```math
\min_i\bigl(\lVert x-c_i\rVert-r_i\bigr)=\delta.
```

Subtract the same constant from every term:

```math
\min_i\bigl(\lVert x-c_i\rVert-r_i-\delta\bigr)=0.
```

Equivalently,

```math
\boxed{\min_i\bigl(\lVert x-c_i\rVert-(r_i+\delta)\bigr)=0.}
```

This is exactly the original F reconstruction with every radius replaced by $r_i+\delta$.

The equivalence concerns the continuous level sets. It assumes the exact minimum field above, one constant positive sigma, and tau strictly between 0 and 1.

Every radius increases by the same absolute distance delta. The radii are not multiplied by a common percentage.

## What the Gaussian transform changes

Applying this decreasing falloff to the minimum sphere distance is equivalent to taking the largest individual sphere falloff:

```math
G(x)=\max_i
\exp\left[-\left(\frac{\max(0,f_i(x))}{\sigma}\right)^2\right].
```

The sphere influences are not summed. At a fixed tau, the exact level set remains the zero level of the enlarged balls. Their boundaries can still meet at sharp creases.

The transform changes the field values used to represent that geometry. Those changed values matter when the field is sampled and interpolated numerically.

## Threshold endpoints

For finite F, the mathematical Gaussian field satisfies

```math
0<G(x)\leq1.
```

At an exterior distance of sigma, its value is

```math
G=\exp(-1)\approx0.367879.
```

It has not reached zero there. It approaches zero farther away; sufficiently small stored values can round to zero in floating point arithmetic.

At tau = 1, the equation G = 1 holds throughout the ball union, including its interior. It does not identify only the boundary. The strict inside test G > 1 has no inside points.

The implementation handles tau = 1 using the limiting F = 0 boundary convention. This is a separately stated choice, rather than a literal extraction using G > 1.

## Grid resolution

Let $L_x,L_y,L_z$ be the side lengths of the input spheres' bounding box. Define its longest side as

```math
L=\max(L_x,L_y,L_z).
```

The requested resolution R sets one common cell width:

```math
h=\frac{L}{R}.
```

All three axes use this same h. The cells are cubic, but the numbers of cells along the axes can differ.

To cover each unpadded side, the nominal cell counts are

```math
N_x=\left\lceil\frac{L_x}{h}\right\rceil,\qquad
N_y=\left\lceil\frac{L_y}{h}\right\rceil,\qquad
N_z=\left\lceil\frac{L_z}{h}\right\rceil.
```

For example, a box measuring 8 by 4 by 2 at R = 128 gives

```math
h=\frac{8}{128}=0.0625.
```

Before padding, this gives:

| Axis | Cells | Cell width |
|---|---:|---:|
| x | 128 | 0.0625 |
| y | 64 | 0.0625 |
| z | 32 | 0.0625 |

Padding adds cells around the box to contain the extracted surface. The final grid dimensions are therefore larger. An axis containing N cells has N + 1 sampled grid points.

Increasing the radii before constructing the grid also enlarges the bounding box. At the same requested R, that changes h. Consequently, equal resolution settings do not necessarily mean identical grids.

## Marching Cubes interpolation

Each reconstruction runs Marching Cubes on one chosen field and one chosen level. It samples that field at grid corners, classifies the corners as inside or outside, and finds crossings along grid edges.

Let $p_a,p_b$ be the endpoints of a crossing edge. Let t be the fraction of the way from $p_a$ to $p_b$. The interpolated position is

```math
p=p_a+t(p_b-p_a).
```

Let $F_a,F_b$ be the sampled F values at those endpoints. Extracting the equivalent offset level $F=\delta$ gives

```math
t_F=\frac{\delta-F_a}{F_b-F_a}.
```

Let $G_a,G_b$ be the corresponding G values. Extracting $G=\tau$ gives

```math
t_G=\frac{\tau-G_a}{G_b-G_a}.
```

Marching Cubes assumes the sampled values change linearly between the endpoints. The Gaussian transform is nonlinear, so these two interpolation fractions generally differ.

On an identical grid, F < delta and G > tau give the same corner classifications in exact arithmetic. With the fixed lookup table used here, that gives the same triangle connections before cleanup. The vertex positions can still differ. Floating point rounding near the chosen level can also affect classifications.

### An explicit edge example

Consider an edge whose F endpoint values are 0 and sigma. Choose tau = 0.5.

The expansion distance is

```math
\delta=\sigma\sqrt{\ln2}.
```

Interpolating F gives

```math
t_F=\frac{\delta}{\sigma}
   =\sqrt{\ln2}
   \approx0.832555.
```

The G endpoint values are 1 and exp(-1). Interpolating G gives

```math
t_G=\frac{0.5-1}{\exp(-1)-1}
   =\frac{0.5}{1-\exp(-1)}
   \approx0.790988.
```

F interpolation places the vertex about 83.26% along the edge. G interpolation places it about 79.10% along the same edge.

Thus, identical continuous level sets can produce different Marching Cubes vertex positions.

## Controlled sphere comparison

A numerical check used one input sphere of radius 1, centered at the origin, with sigma = 0.2 and tau = 0.5. Both fields were sampled with identical grid origins, cell widths and dimensions.

Both reconstructions target the same radius:

```math
r_{\mathrm{target}}
=1+0.2\sqrt{\ln2}
\approx1.166510922.
```

Maximum vertex radius error is the largest absolute difference between a mesh vertex's distance from the center and the target radius.

Let $V_{\mathrm{mesh}}$ be the positive enclosed mesh volume. The exact target volume and relative volume error are

```math
V_{\mathrm{exact}}=\frac{4\pi}{3}r_{\mathrm{target}}^3,
```

```math
E_V=100\frac{\lvert V_{\mathrm{mesh}}-V_{\mathrm{exact}}\rvert}
               {V_{\mathrm{exact}}}.
```

Vertex difference is the maximum Euclidean distance between corresponding vertices placed on the same grid edge.

| R | F vertex radius error | G vertex radius error | F volume error (%) | G volume error (%) | Vertex difference |
|---|---:|---:|---:|---:|---:|
| 32 | 0.000415 | 0.001245 | 0.1704 | 0.0528 | 0.001245 |
| 64 | 0.000104 | 0.000287 | 0.0426 | 0.0123 | 0.000290 |
| 128 | 0.000026 | 0.000070 | 0.0107 | 0.0031 | 0.000071 |

All corner classifications and triangle connections matched at these three resolutions.

G had lower volume error but higher maximum vertex radius error in this experiment. Its numerical effect depends on the error measure and grid.

## Interpreting the homework comparison

The equivalence comparison pairs F = delta with G = tau. Both target the same continuous level set.

The homework comparison instead uses the original F = 0 surface and an expanded G surface, such as G = 0.5. It therefore includes both radius expansion and differences in numerical interpolation.

Let $L_{\mathrm{model}}$ be the longest side of the original model's bounding box. At sigma = 2% of model size and tau = 0.5, the expansion is

```math
\delta
=0.02L_{\mathrm{model}}\sqrt{\ln2}
\approx0.016651L_{\mathrm{model}}.
```

Each radius increases by approximately 1.665% of model size. Improvements in that comparison cannot be attributed solely to Gaussian interpolation, because the continuous target geometry has also changed.
