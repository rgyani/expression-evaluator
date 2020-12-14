# expression-evaluator

Given a string consisting of parentheses, single digits, and positive and negative signs, convert the string into a mathematical expression to obtain the answer.<br>
For example, given '-1 + (2 + 3)', you should return 4.

For simplicity, you can assume only +, -, *, / and ^ are to be supported

### Solution

Arithmetic Expressions can be written in one of three forms:
* Infix Notation: Operators are written between the operands they operate on, e.g. 3 + 4.
* Prefix Notation: Operators are written before the operands, e.g + 3 4
* Postfix Notation: Operators are written after operands. eg 3 4 +

Infix Notation is easier for humans to understand, but harder for computers to evaluate.<br>
We can convert infix to postfix before evaluating using [Shunting-yard algorithm](https://en.wikipedia.org/wiki/Shunting-yard_algorithm)

Input: 3 + 4 × 2 ÷ ( 1 − 5 ) ^ 2 ^ 3


<table class="wikitable">
<tbody><tr>
<th>Token</th>
<th>Action</th>
<th>Output<br />(in <a href="/wiki/Reverse_Polish_Notation" class="mw-redirect" title="Reverse Polish Notation">RPN</a>)</th>
<th>Operator<br />stack</th>
<th>Notes
</th></tr>
<tr>
<td align="center">3</td>
<td>Add token to output</td>
<td>3</td>
<td></td>
<td>
</td></tr>
<tr>
<td align="center">+</td>
<td>Push token to stack</td>
<td>3</td>
<td align="right">+</td>
<td>
</td></tr>
<tr>
<td align="center">4</td>
<td>Add token to output</td>
<td>3 4</td>
<td align="right">+</td>
<td>
</td></tr>
<tr>
<td align="center">×</td>
<td>Push token to stack</td>
<td>3 4</td>
<td align="right">× +</td>
<td>× has higher precedence than +
</td></tr>
<tr>
<td align="center">2</td>
<td>Add token to output</td>
<td>3 4 2</td>
<td align="right">× +</td>
<td>
</td></tr>
<tr>
<td align="center" rowspan="2">÷</td>
<td>Pop stack to output</td>
<td>3 4 2 ×</td>
<td align="right">+</td>
<td>÷ and × have same precedence
</td></tr>
<tr>
<td>Push token to stack</td>
<td>3 4 2 ×</td>
<td align="right">÷ +</td>
<td>÷ has higher precedence than +
</td></tr>
<tr>
<td align="center">(</td>
<td>Push token to stack</td>
<td>3 4 2 ×</td>
<td align="right">( ÷ +</td>
<td>
</td></tr>
<tr>
<td align="center">1</td>
<td>Add token to output</td>
<td>3 4 2 × 1</td>
<td align="right">( ÷ +</td>
<td>
</td></tr>
<tr>
<td align="center">−</td>
<td>Push token to stack</td>
<td>3 4 2 × 1</td>
<td align="right">− ( ÷ +</td>
<td>
</td></tr>
<tr>
<td align="center">5</td>
<td>Add token to output</td>
<td>3 4 2 × 1 5</td>
<td align="right">− ( ÷ +</td>
<td>
</td></tr>
<tr>
<td align="center" rowspan="2">)</td>
<td>Pop stack to output</td>
<td>3 4 2 × 1 5 −</td>
<td align="right">( ÷ +</td>
<td>Repeated until "(" found
</td></tr>
<tr>
<td>Pop stack</td>
<td>3 4 2 × 1 5 −</td>
<td align="right">÷ +</td>
<td>Discard matching parenthesis
</td></tr>
<tr>
<td align="center">^</td>
<td>Push token to stack</td>
<td>3 4 2 × 1 5 −</td>
<td align="right">^ ÷ +</td>
<td>^ has higher precedence than ÷
</td></tr>
<tr>
<td align="center">2</td>
<td>Add token to output</td>
<td>3 4 2 × 1 5 − 2</td>
<td align="right">^ ÷ +</td>
<td>
</td></tr>
<tr>
<td align="center">^</td>
<td>Push token to stack</td>
<td>3 4 2 × 1 5 − 2</td>
<td align="right">^ ^ ÷ +</td>
<td>^ is evaluated right-to-left
</td></tr>
<tr>
<td align="center">3</td>
<td>Add token to output</td>
<td>3 4 2 × 1 5 − 2 3</td>
<td align="right">^ ^ ÷ +</td>
<td>
</td></tr>
<tr>
<td align="center"><i>end</i></td>
<td>Pop entire stack to output</td>
<td>3 4 2 × 1 5 − 2 3 ^ ^ ÷ +</td>
<td></td>
<td>
</td></tr></tbody></table>


The same algorithm can be modified so that it outputs the result of the evaluation of expression instead of a queue. The trick is using two stacks instead of one, one for operands, and one for operators.

```
1. While there are still tokens to be read in,
   1.1 Get the next token.
   1.2 If the token is:
   1.2.1 A number: push it onto the value stack.
   1.2.2 A variable: get its value, and push onto the value stack.
   1.2.3 A left parenthesis: push it onto the operator stack.
   1.2.4 A right parenthesis:
   1 While the thing on top of the operator stack is not a
   left parenthesis,
   1 Pop the operator from the operator stack.
   2 Pop the value stack twice, getting two operands.
   3 Apply the operator to the operands, in the correct order.
   4 Push the result onto the value stack.
   2 Pop the left parenthesis from the operator stack, and discard it.
   1.2.5 An operator (call it thisOp):
   1 While the operator stack is not empty, and the top thing on the
   operator stack has the same or greater precedence as thisOp,
   1 Pop the operator from the operator stack.
   2 Pop the value stack twice, getting two operands.
   3 Apply the operator to the operands, in the correct order.
   4 Push the result onto the value stack.
   2 Push thisOp onto the operator stack.
2. While the operator stack is not empty,
   1 Pop the operator from the operator stack.
   2 Pop the value stack twice, getting two operands.
   3 Apply the operator to the operands, in the correct order.
   4 Push the result onto the value stack.
3. At this point the operator stack should be empty, and the value
   stack should have only one value in it, which is the final result.
```

It should be clear that this algorithm runs in linear time – each number or operator is pushed onto and popped from Stack only once. 
