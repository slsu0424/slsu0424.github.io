---
layout: post
title:  "Exploring LLMs for Social Determinants of Health (SDoH)"
author: sandy
categories: [ healthcare, AI, ChatGPT]
image: assets/images/2024-08/gabin-vallet-unsplash_resize.jpg
featured: false
hidden: false
last_modified_at: 2025-02-26
---

Lately, I have been reading quite a bit about environmental factors affecting healthcare, or what is more commonly known social determinants of health (SDoH).  These include one's financial situation, social circles, and the places they live, work, and play.  Interestingly, I came across a 2016 research paper from the [American Journal of Preventitive Medicine](https://www.sciencedirect.com/science/article/abs/pii/S0749379715005140), which highlights these non-clinical factors as playing a much bigger role in an individual's health outcome than perhaps utilizing healthcare services themselves.  What if we lived in a world where our healthcare services were optimized for the sickest of the sick? In this post, I take a brief look at our government's initiative for health and promising research using LLMs to mine for SDoH from EHRs.

## White House Healthy People 2030
Did you know that every decade since 1979, the US Department of Health and Human Services (HHS) puts out an iteration of the Health People Initiative?  The initiative aims to create a healthier nation and track data-driven outcomes to measure progress towards that goal.  In 2020, HHS released the 5th iteration - Healthy People 2030.  

By background, SDoH has remained a core concept of the Healthy People since its inception, with its definition and importance evolving over time.  According to authors from the HHS [Office of Disease Prevention and Health Promotion (ODPHP)](https://pmc.ncbi.nlm.nih.gov/articles/PMC8478310), Healthy People 2020 included a goal to address social and physical environments that influence health - a clear commitment to reducing health inequities.  SDoH were defined as “the conditions in the environments where people are born, live, learn, work, play, worship, and age that affect a wide range of health, functioning, and quality-of-life outcomes and risks.”

Here are the 5 social determinants of health domains used in Healthy People:

![SDOH](/assets/images/2024-08/sdoh.png){:.centered}

So what's new for 2030?  SDoH is now a [core priority](https://odphp.health.gov/healthypeople/priority-areas), with health disparaties brought to light during the COVID-19 pandemic.  According to the [Journal of Public Health Management & Practice](https://journals.lww.com/jphmp/Fulltext/2021/11001/Healthy_People_2030__A_Call_to_Action_to_Lead.5.aspx), the pandemic highlighted how many Americans fell into a high-risk category for COVID-19 due to underlying health conditions.  Not surprisingly, this is a reflection of disparities in health, access to care, and underlying social determinants.  

I think that a SDoH focus is a step in the right direction, as increasing disease burden continues to affect healthcare stakeholders across the spectrum. 

## Mining EHRs for SDoH evidence
Given the increased focus on SDoH post-pandemic, I think this is an exciting time for using AI with SDoH.  A few research papers have explored using large language models (LLMs) with social determinants of health. For example, this January 2024 study published in [NPJ Digital Medicine](https://arxiv.org/abs/2308.06354) explored extracting SDoH from free text in EHRs.  The results indicate that fine-tuned language models (i.e., retraining pre-trained models for a specialized task) were capable of extracting SDoH from clinical text.  This is a promising application of LLMs, by improving real-world evidence of SDoH.

Another paper by researchers at [Emory University](https://arxiv.org/html/2407.09688) sought to answer the question "Can LLMs streamline integration of SDOH for clinical studies by performing accurate, automatic, generalizable annotation of SDOH variables?"  The team looked at using LLMs to annotate >700 SDoH variables from multiple data sources (including EHRs). The incorporation of these additional variables improved model performance for predicting 30-day hospital readmission.  By using LLMs to automate annotation, they were able to avoid laborious manual annotation - a potentially huge time saver.

## Conclusion
In this post, I reviewed SDoH as a key set of non-clinical factors affecting health outcomes.  Such health disparities surfaced during the COVID-19 pandemic, and became an area of focus for the White House Healthy People 2030 initiative.  Recent AI advances, particularly LLMs, have demonstrated the ability to mine data sources for SDoH evidence.  I remain bullish that these technologies can generate deeper insights and tailored treatments for at-risk patients.

## References
+ <https://www.sciencedirect.com/science/article/abs/pii/S0749379715005140>
+ <https://pmc.ncbi.nlm.nih.gov/articles/PMC8478310>
+ <https://odphp.health.gov>
+ <https://odphp.health.gov/healthypeople/priority-areas>
+ <https://journals.lww.com/jphmp/Fulltext/2021/11001/Healthy_People_2030__A_Call_to_Action_to_Lead.5.aspx>
+ <https://arxiv.org/abs/2308.06354>
+ <https://arxiv.org/html/2407.09688>
