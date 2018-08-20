---
layout:		post
title: 		Pytorch Gotchas
date: 		2018-08-19
categories:	pytorch
---

## Autograd
### No gradient for non-leaf variables by default
One can call `.backward()` on a tensor to compute the gradient of the leaf variables. To save memory, gradients are not computed for non-leaf variables.

{::options parse_block_html="true" /}
<details>
{::options parse_block_html="false" /}

~~~python
import torch
x = torch.ones(1, requires_grad=True)
y = x * 2
z = y * 2

# computes dz/dx for x = 1, because x is a leaf variable
z.backward()
print(x.grad)
print(y.grad)
~~~
Output:
```
tensor([4.])
None
```

To force gradient computation for `y`:
~~~python
y.retain_grad()
~~~
</details>

### Gradients are accumulated
Everytime `.backward()` is called, gradients are <u>added</u> to the `.grad` attribute of the relevant variables.

{::options parse_block_html="true" /}
<details>
{::options parse_block_html="false" /}

~~~python
x = torch.ones(1, requires_grad=True)
y = x * 2
z = x * 3

y.backward()
print(x.grad)

z.backward()
print(x.grad)
~~~
Output:
```
tensor([2.])
tensor([5.])
```

To reset the gradient on a variable:
~~~python
x.grad.zero_()
~~~
</details>
## References
1. <a href="https://discuss.pytorch.org/t/why-cant-i-see-grad-of-an-intermediate-variable/94/2">No gradient for intermediate variable</a>
2. <a href="https://medium.com/@zhang_yang/how-pytorch-tensors-backward-accumulates-gradient-8d1bf675579b">Tensor gradients are accumulated</a>
