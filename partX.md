## A Turtorial to Algorithms (part X)
### How to Estimate The Time-Consuming of An Algorithm
在分析一个算法消耗的时间的时候，假设每一行语句执行消耗常数时间，最后得到一个通式，比如a*n^2 + b*n + c, 这是一个多项式，再对其进行进一步的简化，我们找出其中对算法时间影响最大的一项，也就是n^2, 另外附加一个标记得到O(n^2)这样一种形式来近似表示一个算法消耗的时间，这样我们就得到了一个相对有效而且简介的方式来表示算法消耗的时间，为下面进行的一些算法时间分析做好了准备。 
### symbol table structure and usage
```python
class SemanticProcessor:
  def __init__():
    self._identifiers = {}
    self._declarations = {}
    self._open_scope_stack = []

  def enter_scope():
    binding_list = []
    self._open_scope_stack.append(binding_list)

    #  in traverse progress
    if node is Declaration:
      decl_id = add_declaraction(node)
      self._identifiers[node.name].binding_stack.append(decl_id)

    #  replace all identifier pointer by decl_id

  def leave_scope():
    for x in self._open_scope_stack[-1]:
      # pop from identifier binding stack
    self._open_scope_stack.pop()
```
