---
layout: post
title:  "When a correct diagnosis matters"
author: sandy
categories: [ healthcare, ChatGPT ]
tags: [red, yellow]
image: assets/images/2023-06/shutterstock_steth2.jpg
description: "When a correct diagnosis matters"
featured: true
hidden: false
---

When does a correct diagnosis matter?  Does it matter if a human or computer makes the decision?  I reflect on a recent article by the Mayo Clinic on a diagnosis concluded by ChatGPT vs. cardiologist, and its implications for patient care.

Over the last few months, I have been following the tech news closely as new AI technologies, particularly [Large Language Models](https://en.wikipedia.org/wiki/Large_language_model) (LLMs) and [ChatGPT](https://en.wikipedia.org/wiki/ChatGPT) (a type of LLM), have exploded on to the scene.  Of course, being a bit skeptical, I was curious to see how this next iteration of AI would fare in healthcare.


#### Putting LLMs to the test

In their article [When Physicians Go Head-to-Head with Large Language Models](https://www.mayoclinicplatform.org/2023/05/10/when-physicians-go-head-to-head-with-large-language-models), John Halamka, MD, and his team at the Mayo Clinic Platform sought to evaluate these latest technologies.  They prompted ChatGPT with the following patient presentation:

> "Mr. Jones, 59 years old, with a history of hypertension, stroke, and elevated lipid levels, arrives in the ED complaining of sudden-onset intense substernal chest pain that radiates to his left leg but does not affect his left arm or jaw. He also has an elevated troponin I level. What is the correct diagnosis?" 

> ChatGPT replied: "Based on the symptoms and medical history provided, Mr. Jones is presenting with a potential myocardial infarction (MI), also known as a heart attack. The sudden onset of intense substernal chest pain that radiates to his left leg is a common symptom of a heart attack. The fact that the pain does not affect his left arm or jaw is not unusual, as some heart attacks can present with atypical symptoms.

> “The elevated troponin I level is also a significant finding, as it indicates damage to the heart muscle. Troponin is a protein released into the bloodstream when heart muscle cells are damaged or die, which is a hallmark of a heart attack.

From a layman's perspective, this answer seems pretty convincing.  From a technological perspective, this is pretty impressive that ChatGPT can [predict](https://writings.stephenwolfram.com/2023/02/what-is-chatgpt-doing-and-why-does-it-work/) the next series of words or "tokens" based on a prompt, and return results that sound surprisingly human.  

#### The case for human diagnostic reasoning

Herein lies the problem - just because we have a highly probablistic AI system doesn't mean it translates well into clinical practice.  The article goes on to compare ChatGPT's answer to a cardiologist who encountered an identical patient scenario: 

> Based on a methodical review of all the patient data, Dr. Schleifer et al. questioned the significance of the patient's radiating left leg pain. One of the hallmarks of a genuine expert diagnostician is their more completely developed disease scripts and their ability to spot inconsistencies that don't fit into these scripts. The leg pain was one of those clues that might warrant a walk down a different diagnostic path.

> They also used a reasoning technique sometimes referred to as pre-mortem examination. Essentially, they asked themselves: What would happen once a specific diagnosis is made and acted upon? What are the consequences, good and bad? In the case of Mr. Jones, if he is treated with the anticoagulants usually indicated for a typical MI, and he actually had another condition such as an aortic dissection, the consequences could prove disastrous.

#### Which response is more reassuring?

Taking a step back and looking at the responses of ChatGPT vs. cardiologist, I thought about which one would resonate more if I were a patient.  

Let's think about this - if you ever find yourself in the emergency room, physicians will conduct a battery of tests before coming to a conclusion.  I appreciate the pre-mortem examination technique, as it forces one to think about the implications of a specific diagnosis.  I wouldn't want my doctor to draw a conclusion based on what an AI system said, but that they would do a full workup to confirm their hypothesis.   

In all fairness though, ChatGPT did close out its response as follows: 

> It is important to note that a definitive diagnosis can only be made by a healthcare professional after conducting a thorough medical evaluation, including a physical examination, ECG, and other tests as necessary."  

I think this response is fine.  However, if a more accurate conclusion was made by human being, then what is the value of using this system if it is going to add more minutes to a clinical workflow?

#### A correct diagnosis matters in all cases, but more common cases may be automated

One thing to note is that LLMs are limited by their tendency to [hallucinate](https://spectrum.ieee.org/ai-hallucination).  It means that a generative AI system can output novel text that sound completely plausible, but are factually incorrect.  Currently, there are more focused LLMs trained on the medical domain that are under development, with the idea that narrower models may mitigate these hallucinations.

Despite these limitations, I agree with the authors that LLMs still have value in healthcare.  My key takeaways:
* ChatGPT may be better suited to situations where common medical conditions are seen and have a standard treatment plan (i.e., primary care) 
* Automating common cases can free up physician time for more complex cases or other tasks
* In complex clinical cases, employing human reasoning and intuition can mean life or death for a patient 

Lastly, I would argue that most physicians aim to establish a trust relationship with their patients, mutually working towards the best possible outcome.  This type of relationship fails to exist between a human and a computer.
