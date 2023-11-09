---
title: Example of diagrams, math and source highlighting 
tags: [blog, tutorial]
mermaid: true
mathjax: true
---

* toc
{:toc}

## Enable mermaid rendering

reference [https://jackgruber.github.io/2021-05-09-Embed-Mermaid-in-Jekyll-without-plugin/](https://jackgruber.github.io/2021-05-09-Embed-Mermaid-in-Jekyll-without-plugin/)

With `mermaid: true` you can now enable the rendering of mermaid diagrams in your post or page.
This has the advantage that the relatively large `mermaid.min.js` is only loaded when it is needed.

## Writing a mermaid diagram

[https://mermaid.js.org/syntax/examples.html](https://mermaid.js.org/syntax/examples.html)

You can now write a mermaid diagram like this in your site or posts.

<pre>
```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```  
</pre> 

Which are rendered automatically

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

Entity relationship diagram 

<pre>
```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ LINE-ITEM : contains
    CUSTOMER }|..|{ DELIVERY-ADDRESS : uses
```  
</pre> 

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ LINE-ITEM : contains
    CUSTOMER }|..|{ DELIVERY-ADDRESS : uses
```

Neural network
```mermaid
graph LR
subgraph Input Layer
I1((I1))
I2((I2))
I3((I3))
end
subgraph Hidden Layer 1
H11((H11))
H12((H12))
H13((H13))
H14((H14))
H15((H15))
H16((H16))
end
subgraph Hidden Layer 2
H21((H21))
H22((H22))
H23((H23))
H24((H24))
H25((H25))
H26((H26))
H27((H27))
end
subgraph Output Layer
O1((O1))
O2((O2))
O3((O3))
O4((O4))
end
I1 -->|w11| H11
I1 -->|w12| H12
I1 -->|w13| H13
I1 -->|w14| H14
I1 -->|w15| H15
I1 -->|w16| H16
I2 -->|w21| H11
I2 -->|w22| H12
I2 -->|w23| H13
I2 -->|w24| H14
I2 -->|w25| H15
I2 -->|w26| H16
I3 -->|w31| H11
I3 -->|w32| H12
I3 -->|w33| H13
I3 -->|w34| H14
I3 -->|w35| H15
I3 -->|w36| H16
H11 -->|v11| H21
H11 -->|v12| H22
H11 -->|v13| H23
H11 -->|v14| H24
H11 -->|v15| H25
H11 -->|v16| H26
H11 -->|v17| H27
H12 -->|v21| H21
```
smaller neural network 
```mermaid
graph LR
    subgraph Input_Layer
        I1(Input 1)
        I2(Input 2)
        I3(Input 3)
    end

    subgraph Hidden_Layer_1
        H1(Hidden 1)
        H2(Hidden 2)
        H3(Hidden 3)
        H4(Hidden 4)
    end

    subgraph Hidden_Layer_2
        H5(Hidden 5)
        H6(Hidden 6)
        H7(Hidden 7)
    end

    subgraph Output_Layer
        O1(Output 1)
        O2(Output 2)
    end

    I1 --> H1
    I1 --> H2
    I1 --> H3
    I1 --> H4
    I2 --> H1
    I2 --> H2
    I2 --> H3
    I2 --> H4
    I3 --> H1
    I3 --> H2
    I3 --> H3
    I3 --> H4

    H1 --> H5
    H1 --> H6
    H1 --> H7
    H2 --> H5
    H2 --> H6
    H2 --> H7
    H3 --> H5
    H3 --> H6
    H3 --> H7
    H4 --> H5
    H4 --> H6
    H4 --> H7

    H5 --> O1
    H5 --> O2
    H6 --> O1
    H6 --> O2
    H7 --> O1
    H7 --> O2
```

Large fully connected neural network 
```mermaid
graph TD
    subgraph Input_Layer
        style Input_Layer fill:#D3D3D3,stroke:#333,stroke-width:2px,stroke-dasharray:5,5
        I1((Input 1))
        I2((Input 2))
        I3((Input 3))
    end

    subgraph Hidden_Layer_1
        style Hidden_Layer_1 fill:#D3D3D3,stroke:#333,stroke-width:2px,stroke-dasharray:5,5
        H1((Hidden 1))
        H2((Hidden 2))
        H3((Hidden 3))
        H4((Hidden 4))
        H5((Hidden 5))
        H6((Hidden 6))
    end

    subgraph Hidden_Layer_2
        style Hidden_Layer_2 fill:#D3D3D3,stroke:#333,stroke-width:2px,stroke-dasharray:5,5
        H7((Hidden 7))
        H8((Hidden 8))
        H9((Hidden 9))
        H10((Hidden 10))
        H11((Hidden 11))
        H12((Hidden 12))
        H13((Hidden 13))
    end

    subgraph Output_Layer
        style Output_Layer fill:#D3D3D3,stroke:#333,stroke-width:2px,stroke-dasharray:5,5
        O1((Output 1))
        O2((Output 2))
        O3((Output 3))
        O4((Output 4))
    end

    I1 --> H1
    I1 --> H2
    I1 --> H3
    I1 --> H4
    I1 --> H5
    I1 --> H6
    I2 --> H1
    I2 --> H2
    I2 --> H3
    I2 --> H4
    I2 --> H5
    I2 --> H6
    I3 --> H1
    I3 --> H2
    I3 --> H3
    I3 --> H4
    I3 --> H5
    I3 --> H6

    H1 --> H7
    H1 --> H8
    H1 --> H9
    H1 --> H10
    H1 --> H11
    H1 --> H12
    H1 --> H13
    H2 --> H7
    H2 --> H8
    H2 --> H9
    H2 --> H10
    H2 --> H11
    H2 --> H12
    H2 --> H13
    H3 --> H7
    H3 --> H8
    H3 --> H9
    H3 --> H10
    H3 --> H11
    H3 --> H12
    H3 --> H13
    H4 --> H7
    H4 --> H8
    H4 --> H9
    H4 --> H10
    H4 --> H11
    H4 --> H12
    H4 --> H13
    H5 --> H7
    H5 --> H8
    H5 --> H9
    H5 --> H10
    H5 --> H11
    H5 --> H12
    H5 --> H13
    H6 --> H7
    H6 --> H8
    H6 --> H9
    H6 --> H10
    H6 --> H11
    H6 --> H12
    H6 --> H13

    H7 --> O1
    H7 --> O2
    H7 --> O3
    H7 --> O4
    H8 --> O1
    H8 --> O2
    H8 --> O3
    H8 --> O4
    H9 --> O1
    H9 --> O2
    H9 --> O3
    H9 --> O4
    H10 --> O1
    H10 --> O2
    H10 --> O3
    H10 --> O4
    H11 --> O1
    H11 --> O2
    H11 --> O3
    H11 --> O4
    H12 --> O1
    H12 --> O2
    H12 --> O3
    H12 --> O4
    H13 --> O1
    H13 --> O2
    H13 --> O3
    H13 --> O4

```

definition of finite automata 
A
```mermaid
graph TD
    q0 -->|a| q0
    q0 -->|b| q1
    q1 -->|a| q2
    q1 -->|b| q2
    q2 -->|a| q2
    q2 -->|b| q2
    q0[(>q0)]
    q1[(q1)]
    q2[(q2)]
```

B
```mermaid
graph TD
    q0 -->|a| q1
    q0 -->|b| q2
    q1 -->|a| q3
    q1 -->|b| q0
    q2 -->|a| q0
    q2 -->|b| q2
    q3 -->|a| q3
    q3 -->|b| q3
    q0[(>q0)]
    q1[(q1)]
    q2[(q2)]
    q3[(q3)]
```

## Math Symbols with MathJax

reference [https://jojozhuang.github.io/tutorial/jekyll-math-symbols-with-mathjax/](https://jojozhuang.github.io/tutorial/jekyll-math-symbols-with-mathjax/)

### Showing Math Symbols on Web Page
The support of displaying math symbols with html tags is limited. Though you can use UTF-8 to display some special characters, it is hard to remember them and it is inconvenient to use.

Title                   | Formula                 | HTML Tag
------------------------|-------------------------|--------------------------------------
Square                  | n<sup>2</sup>           | `n<sup>2</sup>`
Square Root             | &radic;, &#8730;        | `&radic;`, or `&#8730;`
Summary                 | &sum;, &#8721;          | `&sum;`, or `&#8721;`

* [UTF-8 Mathematical Operators](https://www.w3schools.com/charsets/ref_utf_math.asp)

### MathJax
[MathJax](https://www.mathjax.org/) is a cross-browser JavaScript library that displays mathematical notation in web browsers.

With `mathjax: true` you can now enable the rendering of mathjax expressions in your post or page.


1) Example One:
Construct the formula with following markdown.
```raw
$a^2 + b^2 = c^2$
```
Then you will get the formula as follows.

$a^2 + b^2 = c^2$

2) Example Two:
Construct the formula with following markdown.
```raw
$ x = {-b \pm \sqrt{b^2-4ac} \over 2a} $
```
Then you will get the formula as follows.

$ x = {-b \pm \sqrt{b^2-4ac} \over 2a} $

3) Example Three:
Construct the formula with following markdown.
```raw
$$\begin{eqnarray}
x' &=& &x \sin\phi &+& z \cos\phi \\
z' &=& - &x \cos\phi &+& z \sin\phi \\
\end{eqnarray}$$
```
Then you will get the formula as follows.

$$\begin{eqnarray}
x' &=& &x \sin\phi &+& z \cos\phi \\
z' &=& - &x \cos\phi &+& z \sin\phi \\
\end{eqnarray}$$

Here are some notes about the above example:
* The inline formula is between `$ ... $`.
* The display formula is between `$$ ... $$`.
* You can use the math envrionment directly, for example, `\begin{equation}...\end{equation}` or `\begin{align}...\end{align}`.
* Whenever in the inline math or display math, the star character `*` must be escaped.
* In the multi-lines display math, the line break symbol double-backslash `\\` should be escaped, i.e., use four backslash `\\\\`.
* If you found error while typesetting math formula, try to escape some special characters.

## Source code highlighting
some of the supported language identifiers : **java, javascript, html, c, cpp, ruby, xml, yaml, json, clojure, smalltalk**

<pre>
```java
    // A method that sorts an array of integers using bubble sort algorithm
    public static void bubbleSort(int[] array) {
        // Get the length of the array
        int n = array.length;
        // Loop through the array from the first element to the second last element
        for (int i = 0; i < n - 1; i++) {
            // Loop through the array from the first element to the last element that is not sorted yet
            for (int j = 0; j < n - i - 1; j++) {
                // Compare the current element with the next element
                if (array[j] > array[j + 1]) {
                    // Swap them if the current element is larger than the next element
                    int temp = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = temp;
                }
            }
        }
    }
```
</pre>
```java
    // A method that sorts an array of integers using bubble sort algorithm
    public static void bubbleSort(int[] array) {
        // Get the length of the array
        int n = array.length;
        // Loop through the array from the first element to the second last element
        for (int i = 0; i < n - 1; i++) {
            // Loop through the array from the first element to the last element that is not sorted yet
            for (int j = 0; j < n - i - 1; j++) {
                // Compare the current element with the next element
                if (array[j] > array[j + 1]) {
                    // Swap them if the current element is larger than the next element
                    int temp = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = temp;
                }
            }
        }
    }
```

## Displaying line numbers for highlighted code 

{% raw %}
<pre>
{% highlight java linenos %}
    public class HelloWorld {
        public static void main(String[] args) {
            System.out.println("Hello World!");
        }
    }
{% endhighlight %}
</pre>
{% endraw %}



{% highlight java linenos %}
    public class HelloWorld {
        public static void main(String[] args) {
            System.out.println("Hello World!");
        }
    }
{% endhighlight %}


