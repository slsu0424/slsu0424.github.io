---
layout: post
title:  "Relevance of transformer models in healthcare"
author: sandy
categories: [ healthcare, EHR, AI ]
image: assets/images/2024-04/iStock-1937350031_license_resize.jpg
featured: false
hidden: false
---

Recently, healthcare workers openly voiced concerns about the rush to adopt [generative AI in healthcare](https://aiin.healthcare/topics/artificial-intelligence/overheard-around-kaiser-nurses-protest-over-ai-healthcare), pitting the novel technology against the humanization of care.  Healthcare is arguably at an inflection point where innovation is needed, but what happens to the patients and those who care for them?  As a business leader or technologist, it is important to ask:  In what ways are these new AI models superior to their predecessors?  What problem can these AI models actually solve?  In this post, I take a deeper look into transformer models and their potential utility in healthcare.

## A primer on transformer models
There is no shortage of resources to learn about transformers such as the Generative Pre-trained Transformer (GPT).  I will not go into the details here, but provide links to blogs that best illustrate key concepts.  

At a high-level, transfomer models make use of the [self-attention mechanism](https://arxiv.org/abs/1706.03762), which is a ML technique that calculates the importance between different pairs of words in an input sequence, and processes them in parallel.  This enables the model to more accurately focus on the most relevant words when generating new output.  Check out these technical blogs for further details:
- [Attention to improve machine translation models](https://jalammar.github.io/visualizing-neural-machine-translation-mechanics-of-seq2seq-models-with-attention)
- [Transformers (illustrated)](https://jalammar.github.io/illustrated-transformer)
- [Self-attention & multi-head attention](https://devennn.github.io/2020/self-and-multihead-attention) 

Here is a simplified [visual](https://towardsdatascience.com/illustrated-guide-to-transformers-step-by-step-explanation-f74876522bc0) of how attention mechanism works, and how it compares to LTSM and RNN architectures:

![AttentionMechanism](/assets/images/2024-04/atm.png)

![LSTM](/assets/images/2024-04/lstm.png)

![RNN](/assets/images/2024-04/rnn.png)

As shown above, this technique overcomes the limitation of previous NLP models which ran into the problem of having short-term memory.  As a result, these AI models can more accurately retain the meaning of longer input sequences when generating new output. 

## Are transformers useful in healthcare?
Now that we see how the transformer model has succeeeded over previous NLP models, let's take a look at how it is being applied in healthcare.  I should caveat this section by saying that this field is evolving very rapidly, so what has been published in recent months is likely be challenged and revised.  My goal here is to provide a small snapshot of studies I found compelling.  

In this February 2024 paper, [Transformer Models in Healthcare: A Survey and Thematic Analysis of Potentials, Shortcomings and Risks](https://link.springer.com/article/10.1007/s10916-024-02043-5), authors from Europe highlighted some of the key benefits and challenges with these models:

![Fig2](/assets/images/2024-04/Fig2.png)

It should not come as a surprise that one of the biggest benefits would be improved operational efficiencies (A1-A4).  While EU counterparts probably face less documentation requirements than the US (as I wrote about in a previous [post](https://slsu0424.github.io/ehrs-us-can-llms-make-significant-impact)), it highlights a main value of these AI systems: synthesizing data quickly.  

However, when it comes to clinical decision-making, one of the biggest challenges is addressing the "black box" phenomenon by which LLMs generates answers.  As highlighted in this [paper](https://bmcmedinformdecismak.biomedcentral.com/articles/10.1186/s12911-024-02459-6#author-information), transfomer models have faced challenges such as navigating ambiguous or complex [clinical scenarios](https://www.medrxiv.org/content/10.1101/2023.02.21.23285886v1), including biases from training data, and the inablity to take into account a patient's social determinants of health.  This recognition has given rise to building domain-specific LLMs, and adding in other diagnostic criteria to provide a more holistic patient profile.

This Stanford [study](https://www.nature.com/articles/s41746-024-01010-1#author-information) set out to address this problem using specialized instructional prompts that mimic the diagnostic reasoning criteria used by physicians.  They compared this technique to [Chain-Of-Thought (CoT)](https://arxiv.org/abs/2201.11903) prompting, to assess whether LLMs could exhibit clinical reasoning.  They found that LLMs can imitate clinical reasoning processes when using the specialized prompts, but still cannot reason like a human.  The authors conclude that the finding is still significant, in that the LLM provides rationale which can be vetted for accuracy by clinicians.

![Fig1](/assets/images/2024-04/Fig1.png)


## Reasons to remain (cautiously) optimistic
In each of these examples, transformer models appear to provide varying levels of utility, with perhaps the greatest potential for administrative tasks such as patient summarization and patient education.  I personally feel that clinical decision-making, or anything that involves human intuition, will never be replaced by a computer.  Some argue that generative AI can be a powerful aid for all healthcare stakeholders.  Others argue it will add unnecessary steps to the clinical workflow.  

Yet, what excites me the most is the overall effort by clinicians, researchers, and informaticists to wrestle with the challenges introduced by this technology.  In my view, these groups must be at the forefront to advocate for the right use of this technology to improve patient care.


## Conclusion
The "gold rush" to adopt generative AI in healthcare is exciting yet problematic.  The transformer architecture has markedly improved NLP performance, giving way to potentially limitless applications in healthcare.  I believe transformer models are best suited for clinical summarization tasks, but I remain skeptical on their utility for clinical decisions.  Perhaps the latter will improve over time with specialized models and frameworks.


## References
+ <https://aiin.healthcare/topics/artificial-intelligence/overheard-around-kaiser-nurses-protest-over-ai-healthcare>
+ <https://arxiv.org/abs/1706.03762>
+ <https://jalammar.github.io/visualizing-neural-machine-translation-mechanics-of-seq2seq-models-with-attention>
+ <https://jalammar.github.io/illustrated-transformer>
+ <https://devennn.github.io/2020/self-and-multihead-attention>
+ <https://link.springer.com/article/10.1007/s10916-024-02043-5>
+ <https://bmcmedinformdecismak.biomedcentral.com/articles/10.1186/s12911-024-02459-6#author-information>
+ <https://www.nature.com/articles/s41746-024-01010-1#author-information>
+ <https://arxiv.org/abs/2201.11903>