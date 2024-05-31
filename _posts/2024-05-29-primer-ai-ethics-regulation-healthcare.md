---
layout: post
title:  "A primer on AI ethics and regulation in healthcare"
author: sandy
categories: [ healthcare, AI, ethics, regulation ]
image: assets/images/2024-05/tingey-injury-law-firm-resize.jpg
featured: false
hidden: false
---

I had not explored much into AI ethics and regulation until recently, when this [article](https://www.politico.com/news/2024/03/24/who-pays-when-your-doctors-ai-goes-rogue-00148447) shed light into the [on-going dilemma](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8452365) of liabilty when AI is used for patient care.  The reason this is important now is because AI tools are proliferating in healthcare more than ever before.  Should physicians ultimately be on the hook?  What role do regulators play?  What would it take to integrate AI tools to become standards of care?  In this post, I review the current discussion around these questions and recommended paths going forward.  

## Who makes the final call when AI is involved?
The aforementioned article states that physicians should be on the hook to understand the technology they are using when it comes to clinical decision-making.  At least, that's the point of view from the technology companies.  Physicians have been put in this unfavorable position in the past, subject to lawsuits when physicians have followed erroneous software recommendations.  What's different this time around is that physicians could also be held accountable if they *did not* make use of AI at all.  Let's say a Large Language Model (LLM) were at a physician's disposal, would it be considered patient negligence if they did not make use of its recommendations?  What recourse does a physician have in this case?

According to Dr. Michelle Mello, a Stanford health law scholar:

>Unregulated tools could be more vulnerable to lawsuits, ... because they aren’t protected by the “preemption doctrine,” which bars some claims against regulated devices on the theory that they’re known to be safe enough and effective enough to have received FDA clearance.

I found this comment striking because to my knowledge, there are no known LLMs that have been [approved](https://www.fdli.org/2023/12/tech-support-fdas-evolving-regulatory-plan-for-drug-and-device-enabling-software) by the FDA:

> To compound FDA’s regulatory challenge in this space, AI/ML capabilities have increased significantly in recent years, and even in recent months, without signs of slowing, due to the development of [LLMs] ... Although FDA has not cleared or approved any LLM software products to date, it will no doubt be a significant item on the regulatory agenda moving forward.

To note, this should not be mistaken for AI-enabled devices that use other deep learning algorithms that have been recently approved by the FDA.  For example, the [Eko Low Ejection Fraction Tool (ELEFT)](https://www.accessdata.fda.gov/scripts/cdrh/cfdocs/cfpmn/pmn.cfm?ID=K233409) was approved for cardiology use on March 24, 2024.  It makes use of deep neural network and convolutional neural network models for classification, per their summary document.

If there is increased liability for using such tools, then there is little incentive for adopting these in real-world settings.

## Ensuring governance for all
If one wants to get a good look at what regulation for AI in healthcare should look like, I found Dr. Mello's [February 2024 testimony](https://www.finance.senate.gov/imo/media/doc/02082024_mello_testimony.pdf) to the U.S. Senate Committee on Finance to be an excellent resource.  In the document, she highlights key learnings from evaluating AI tools:

1.  Not all healthcare organizations have the resources to develop AI review processes.
2. Governance should involve how an AI algorithm works within a clinical workflow.  Who is to ensure that clinical staff uses AI the way that it was intended by software developers?
3. The government should establish standards for using AI tools in healthcare.  However, regulation needs to be agile and should reside within a consensus-building entity (e.g., [Coalition for Health AI](https://jamanetwork.com/journals/jama/fullarticle/2813425)).  Federally regulated programs, such as Medicare, Medicaid, the VA health system, should adhere to these standards.

Lastly, Dr. Mello notes that health insurers also need AI regulation, and points to the example of unregulated AI used by Medicare Advantage plans to [deny care](https://www.statnews.com/2023/03/13/medicare-advantage-plans-denial-artificial-intelligence) for seniors in need.  It highlights the risk when individual circumstances are not taken into account - when the patient should be getting more care than what an algorithm suggests. 

## Conclusion
Throughout this journey in researching the societal effects of AI in healthcare, its become clear that we are navigating choppy waters with the rapid development of these tools.  I believe this innovation is a good thing, and the healthcare industry is finally listening.  I like the idea of consensus-built AI standards, and there should be more scrutiny from every angle.  Its not out of reach to say that AI can improve health outcomes for all, but implementation will require thoughtful investment across the ecosystem. 

## References
+ <https://www.politico.com/news/2024/03/24/who-pays-when-your-doctors-ai-goes-rogue-00148447>
+ <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8452365>
+ <https://www.fdli.org/2023/12/tech-support-fdas-evolving-regulatory-plan-for-drug-and-device-enabling-software>
+ <https://www.accessdata.fda.gov/cdrh_docs/pdf23/K233409.pdf>
+ <https://www.finance.senate.gov/imo/media/doc/02082024_mello_testimony.pdf>
+ <https://jamanetwork.com/journals/jama/fullarticle/2813425>
+ <https://www.statnews.com/2023/03/13/medicare-advantage-plans-denial-artificial-intelligence>