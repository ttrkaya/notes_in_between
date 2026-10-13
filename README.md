Let \(x\) be a position in space, and \(c_i,r_i\) the center and radius of sphere \(i\). The quantity \(\|x-c_i\|\) is the distance from \(x\) to that center.

Our fields are

\[
F(x)=\min_i\bigl(\|x-c_i\|-r_i\bigr),
\]

\[
G(x)=\exp\left[-\left(\frac{\max(0,F(x))}{\sigma}\right)^2\right],
\]

where \(\sigma>0\) is the Gaussian width.

Choose a surface level \(G(x)=\tau\), with \(0<\tau<1\). Inside the balls, G is 1, so this surface must have \(F(x)>0\). We can therefore remove the \(\max\):

\[
\exp\left[-\left(\frac{F(x)}{\sigma}\right)^2\right]=\tau.
\]

Take the natural logarithm:

\[
-\left(\frac{F(x)}{\sigma}\right)^2=\ln\tau.
\]

Since F is positive, take the positive square root:

\[
F(x)=\sigma\sqrt{-\ln\tau}.
\]

Call that constant distance \(\delta\). Substituting the definition of F gives

\[
\min_i\bigl(\|x-c_i\|-r_i\bigr)=\delta.
\]

Subtracting the same constant from every term gives

\[
\boxed{\min_i\bigl(\|x-c_i\|-(r_i+\delta)\bigr)=0.}
\]

That is exactly the original F reconstruction with every radius replaced by \(r_i+\delta\).
