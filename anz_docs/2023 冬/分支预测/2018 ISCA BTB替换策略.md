***A key idea in recent work is that sequences of recently accessed instructions correlate strongly with the likelihoods of block reuse. Training a predictor on control-flow traces based on sampled sets yields a high reuse prediction accuracy**

**key insight:
* PC-based predictors exploit the observation that, if a block in the data cache is accessed by a given PC and becomes dead,other blocks accessed by the same PC in other sets are likely to become dead as wel

方法：
用global path history of instruction address
signature formula:每次访问将PC的最低3位左移进history，再补一位0