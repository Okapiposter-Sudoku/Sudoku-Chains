# A (Mostly Logical) Guide to Sudoku Chains

This guide is my attempt to summarize and explain the theory of *chains* in Sudoku solving. I will formalize all concepts using [*Propositional*](https://en.wikipedia.org/wiki/Propositional_calculus) and [*First-Order Logic*](https://en.wikipedia.org/wiki/First-order_logic), but I will introduce all terms and notation so you as the reader don't need to be familiar with those fields.

## Basic Terms and Concepts

### Cells

I'm assuming that you are already familiar with the basic building blocks of the Sudoku grid, i.e. 81 *cells* each contained in one of nine *rows*, *columns* and 3x3 *boxes*. To address a single cell in the grid, we use two different schemes interchangeably:

  1. **r***X***c***Y* references the cell at the intersection of row *X* and column *Y* in the grid.
  2. **b***A***p***B* references the cell at position *B* inside box *A*, where the positions inside each box are numbered the sae way as the boxes inside the grid:
     ```
     ,---------.
     | 1  2  3 |
     | 4  5  6 |
     | 7  8  9 |
     `---------´
     ```
> *Example:*  
> The cell at the intersection of row 7 and column 8 in the grid can be addressed as either **r7c8** or as **b9p2**.

Groups of cells can be addressed with both schemes by specifying more than one row, column box and/or position. This will reference all cells addressable by picking one of the digits from the first and one of the second group, so **r***UV***c***WX* would address **r***U***c***W*, **r***U***c***X*, **r***V***c***W* and **r***V***c***X*.

> *Example:*  
> The last six cells of the first two rows of the grid can be referenced both as **r12c456789** and as **b23p123456**.

### Candidates

While in the final, solved state of the grid every cell will contain exactly one digit (each obeying the rules of Sudoku), in the process of solving some cells will still be *open*, with no digit de termined yet. In such an open cell **r***X***c***Y*, we call a digit *D* a *candidate* (written as *(D)***r***X***c***Y*) if we haven't excluded the possibility yet that **r***X***c***Y* will contain digit *D* in the finished puzzle. All non-open cells have one single candidate, namely the digit written into the cell.

> *Example:*  
> **(5)r3c4** stands for the *proposition* (i.e., statement which is either true or false) that “In the completed puzzle, the cell **r3c4** will contain digit **5**.”

Groups of candidates can again be specified by writing more than one digit in-between the parentheses at the front of the expression, and candidates can be specified for groups of cells by using the syntax explained above under “Cells”. The meaning of this is that *all* of the digits between the parentheses will occur *somewhere* in the cells specified after the parentheses.

> *Formally:*  
> The expression *(D)C* (where *D* is a set of digits and *C* is a set of cells in either of the schemes defined above) stands for the proposition
> > *(D)C* ≝ “In the completed puzzle, the cells specified by *C* will contain all of the digits *D* somewhere in them.”
> 
> The symbol “≝” defines the expression on the left to have the meaning on the right.

> *Examples:*  
>  1. **(1)r2c3** stands for the proposition “In the completed puzzle, cell **r2c3** will contain digit 1.”
>  2. **(13)b1p45** stands for the proposition “In the completed puzzle, cells **b1p4** and **b1p5** will contain both a 1 and a 3.”
>  3. **(45)r5c123** means “In the completed puzzle, the first three cells of row 5 will contain both a 4 and a 5.”

## Links

Chain-based Sudoku techniques work by establishing relationships between (groups of) candidates in the grid. The two essential basic relationships from which chains are built are called *strong* and *weak links*.

### Strong Link

Two (groups of) candidates *P* and *Q* in the grid are *strongly linked* (sometimes symbolized as double line or an equals sign “*P*=*Q*”) if *P* or *Q* will always be true in the finished grid, *both can't be false at the same time*.

> *Formally:*  
> The logical connective for two propositions which can't both be false is “∨”, the *OR Gate* or *OR Operator*. So “*P*=*Q*” stands for the proposition that at least one of *P* and *Q* will always be true in the finished grid.
> > *P*=*Q* ≝ *P* ∨ *Q*

Now let's look at some examples of *direct* (i.e., not composed out of other links) strong links that can be spotted in the grid.

> *Examples:*  
>  * **Bivalue Strong Link:** If a cell in the grid has only two viable candidates *X* and *Y* left (called a *bivalue cell*), the two candidates are strongly linked (*X*=*Y*) because each cell must contain some digit in the finished grid.
>  * **Bilocal Strong Link:** If there are only two candidates *X* and *Y* for a certain digit inside one *house* (collective name for rows, column and boxes), the two candidates are strongly linked (*X*=*Y*) because every digit must be present in each house of the finished grid.
>  * **Grouped Strong Link:** Actually you can pick any two groups of cells *V* and *W* in the grid (which may or may not overlap) so that V and W together contain all candidates for some digit *D* inside some house *H*. Since *H* must contain a *D* somewhere, (*D*)*V* and (*D*)*W* can't both be false, so (*D*)*V*=(*D*)*W* is a valid strong link.
>  * **ALS Strong Link:** Assume that you find a group *V* of *n* cells inside some house *H* so that the group collectively only contains candidates for *n*+1 different digits *D*. We can't repeat the same digit inside house *H*, so all but one digits in *D* will need to be inside *V* in the finished puzzle. So if *X* is some digit in *D* and *D'* is the set of all digits in *D* except for *X*, then either *X* must be in *V* or if it isn't then *V* must be filled with all *n* remaining digits in *D'*. So (*X*)*V*=(*D'*)*V* is a valid strong link.

There are more types of direct strong links (e.g. *Almost-Fish Strong Links*), which I will not discuss here. The principles stay the same, and all stron links can be used in the chains discussed below.

### Weak Link

Two (groups of) candidates *P* and *Q* in the grid are *weakly linked* (sometimes symbolized as single line or a dash “*P*-*Q*”) if *P* and *Q* can never both be true in finished grid, *at least one of them will always be false*.

> *Formally:*  
> The logical connective for two propositions which can't both be true is “⊼”, the *NAND Gate* or *NAND Operator* (an abbreviation for *not-and*). So “*P*-*Q*” stands for the proposition that at least one of *P* and *Q* will always be false in the finished grid.
> > *P*-*Q* ≝ *P* ⊼ *Q*

Now let's look again at examples of direct weak links:

> *Examples:*  
>  * **Cell Weak Link:** Each cell in the finished grid will contain exactly one digit. So for every cell *V* and every pair of digits *X* and *Y* there is a weak link (*X*)*V*-(*Y*)*V* because the single cell can't contain both digits. Note that both candidates can still be false if the cell still has viable candidates other than *X* or *Y*.
>  * **House Weak Link:** In each house *H* there will be exactly one cell which contains any digit *X* in the finished grid. So if you look at two non-overlapping groups of cells *V* and *W* inside *H*, there can't be an *X* in both of them. There is a weak link (*X*)*V*-(*X*)*W* between the candidates for *X* in the two groups of cells.

There are again more types of direct weak links (e.g. *AHS Weak Links*), which I'm not discussing here.

### A Note on Dual Strong and Weak Links

In some common situations two candidates in the grid are both strongly *and* weakly linked, which I will call a *dual link*. The two canonical examples for this are

 1. the two candidates of a *bivalue cell*, which are linked both through a *bivalue strong link* and a *cell weak link*, and
 2. the two candidates of a *bilocal digit* inside a house, which are linked both through a *bilocal strong link* and through a *house weak link*.

> *Formally:*  
> If two (sets of) candidates *P* and *Q* are both strongly *and* weakly linked (“∧” is the logical “AND Operator”) then exactly one of them is true and the other is false. This relationship is represented by “⨁”, the “XOR Operator”. 
> > (*P*=*Q* ∧ *P*-*Q*)  ⇔  (*P* ⨁ *Q*)
> 
> where “⇔” signifies logical equivalence between the left and right side.

Historically some authors used the term “strong link” only for dual links and did not have a separate name for pure strong links (like ALS strong links or overlapping grouped strong links) which are not also weakly linked. This leads to claims like “every strong link is also weak” or “you can always use a strong link where a weak link is needed”, which are not correct with my usage of those terms. To bridge this gap in terminology some authors now use “strong inference” and “weak inference” for the conclusions which can be made from links and say that both strong and weak inferences can be drawn from a strong link.

In my opinion this historical usage of “strong link” is confusing and does not mix well with more advanced links like ALS strong links and chain terminology. If an ALS strong link is not a strong link but an inference, we can't say any more that Sudoku chains are constructed from links. For this reason I'm not trying to stay consistent with the old terminology, so keep that in mind when reading other Sudoku content.


## Short Combinations of Links

Before jumping into long chains, let's first think about what we can deduce from short combinations of two to three links. In the tradition of [Eureka Notation](http://sudopedia.enjoysudoku.com/Eureka.html) we will write there combinations in sequence, so “A-B=C” means that there is a weak link between A and B and also a strong link between B and C.

### Weak + Strong

If there are three (groups of) candidates *U*, *V* and *W* in the grid so that there is a weak link between *U* and *V* and a strong link between *V* and *W*, *U*-*V*=*W*, then we know that

 * if *U* is true then *V* is false (because of *U*-*V*), so *W* is true (because of *V*=*W*) and
 * if *U* is false then we know nothing about *V* or, by extension, *W*.

Looking from the other side,

 * if *W* is true then we know nothing about *V* or, by extension, *U* and
 * if *W* is false then *V* is true (because of *V*=*W*), so *U* is false (because of *U*-*V*).

So from the links we can see that *U* being true makes *W* true and, inversely, *W* being false makes *U* false. No inference can be made for the other truth value in each direction. This relationship is called an “implication”.

> *Formally:*  
> If *A* and *B* are two propositions, then “*A* → *B*” (read as “*A* implies *B*” or “if *A* then *B*”) is a proposition which is false if and only if *A* is true **and** *B* is false. It expresses the relationship in which *A* being true will always mean that *B* is also true.

### Weak + Weak, Strong + Strong

\#TODO

### Dual + Dual

\#TODO

## Longer Single-Stranded Chains

### Unidirectional (Forcing Chain)

\# TODO

### Bidirectional (AIC)

\#TODO

## Multiple Strands from a Single Source (FC Moves, Death Blossom, ...)

\#TODO

## Nets

\#TODO


