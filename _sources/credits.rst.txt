*******
Credits
*******


The code within the `*.text` packages is primarily derived from Apache's StringUtils and Commons Text libraries.

In earlier versions of this library, these Apache components were directly included as dependencies. 
However, this often led to complex version conflicts, as other libraries within our projects also relied on `StringUtils`` and `Commons Text`, 
frequently in different versions.

To mitigate these dependency conflicts, 
we decided to remove the direct library dependencies and instead integrate the necessary code snippets directly into our `*.text` packages. 
An additional benefit of this approach is a reduction in the overall library size.

