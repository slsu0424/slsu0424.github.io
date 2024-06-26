---
layout: post
title:  "EHRs in the US: Can LLMs make a significant impact?"
author: sandy
categories: [ healthcare, EHR, AI ]
image: assets/images/2024-02/iStock-815517436_license_resize.png
featured: false
hidden: false
---

Continuing on from my previous post on emerging tech's value in healthcare, I was intrigued to review electronic health record (EHR) adoption in the US and across other countries.  While we should applaud the fact that [96% of US hospitals](https://www.beckershospitalreview.com/ehrs/96-of-us-hospitals-have-ehrs-but-barriers-remain-to-interoperability-onc-says.html) have EHRs as of 2022, fulfilling [policies](https://www.healthcareitnews.com/news/10-years-meaningful-use-major-progress-despite-challenges) set forth by Presidents Bush and Obama, it has produced [frustration](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7647027) among many physicians.  Recently, LLMs have shown promise to alleviate some of this burden by mining EHR data in novel ways.  But is this emerging tech significant enough to improve overall health outcomes?

## EHR use in the US vs. abroad
A 2020 JAMA [study](https://jamanetwork.com/journals/jamainternalmedicine/fullarticle/2774129) set out to compare clinician usage of EHRs in the United States and in other countries.  The report studied EHR metadata of 371 health systems in the US and abroad.  I will focus on the paper's key findings and discussion, and its implications from a technology perspective.

Key findings on US clinicians:
1. Spent more time per day using the EHR
2. Received more system-generated messages
3. Wrote a higher proportion of automatically generated notes
4. Spent more time using the EHR after hours

Discussion:
1. Recent [research](https://www.healthsystemtracker.org/chart-collection/quality-u-s-healthcare-system-compare-countries) has shown that US healthcare organizations do not surpass neighboring countries in most process quality measures during care delivery.  Its unlikely that information technology could help improve these measures significantly.
2. The [Meaningful Use](https://www.ama-assn.org/practice-management/medicare-medicaid/meaningful-use-electronic-health-record-ehr-incentive) incentive program mandated both secure messaging with patients and electronic prescribing.  This has meant additional time spent by US clinicians on handling messages.
3. EHR functions, such as [e-prescribing](https://www.cms.gov/medicare/regulations-guidance/electronic-prescribing), may improve safety and save time in the care delivery process.  Non-US clinicians may write prescriptions manually.
4. US clinicians used EHRs in different ways that are driven by unique workflows or policy.  A multipayer billing environment and Meaningful Use requirements may contribute to clinicians overdocumenting to ensure reimbursable claims or to prevent malpractice lawsuits.  

## Considerations for the technologist
Earlier in my career, I remember talking to a physician friend about what IBM Watson was doing in healthcare.  In a voice of skepticism, they responded "Sounds like a solution looking for a problem".  

I'll never forget that statement.

In my experience, the next big tech idea in healthcare does not always fix problems.  Oftentimes, it introduces new ones.  There has to be a clear and compelling problem that technology can actually solve, and one with a demonstrable ROI.  

The administrative burden of EHRs in the US is a big problem.

Its an area ripe for disruption, especially at a time when hospitals continue to [lose money](https://www.kaufmanhall.com/insights/thoughts-ken-kaufman/numbers-behind-national-hospital-flash-report).  The text summarization capabilites of LLMs make them a compelling solution for today's EHR woes.  I believe there is a fit in the clinicical workflow, and the industry is on the right path to [integrate](https://www.techtarget.com/searchhealthit/feature/How-4-EHR-Vendors-Are-Leveraging-Generative-AI-in-Clinical-Workflows) these new capabilites into EHRs.  

However, there are many other factors that may limit the technology's impact.  For example, LLMs tendency to hallunicate should be of concern to clinicians and patients alike.  The rise of [Retrieval augmented generation (RAG)](https://www.computerworld.com/article/3712789/how-rag-makes-generative-ai-tools-even-better.html) techniques should help mitigate the pitfalls of LLMs.  A close partnership between technology vendors and customers will be extremely important, as will selecting use cases that can produce tangible results.

Technology aside, there are still significant market forces, policies, and existing workflows that must be overcome.  As the JAMA report cites, more must be done at the policy level to address clinician burnout.  I believe emerging tech can help to some extent, but by how much still remains to be seen.  

## Conclusion
This paper was insightful to understand the differences in EHR adoption in the US and abroad.  I was surprised to learn that clinicians in other countries were not subject to the same level of scrutiny in filing medical claims, highlighting the need for change at the policy level.  Thus, despite whatever newest technology is available, there needs to be additional changes to how healthcare is regulated in the US.


## References
+ <https://www.beckershospitalreview.com/ehrs/96-of-us-hospitals-have-ehrs-but-barriers-remain-to-interoperability-onc-says.html>
+ <https://www.healthcareitnews.com/news/10-years-meaningful-use-major-progress-despite-challenges>
+ <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7647027>
+ <https://jamanetwork.com/journals/jamainternalmedicine/fullarticle/2774129>
+ <https://www.healthsystemtracker.org/chart-collection/quality-u-s-healthcare-system-compare-countries>
+ <https://www.ama-assn.org/practice-management/medicare-medicaid/meaningful-use-electronic-health-record-ehr-incentive>
+ <https://www.cms.gov/medicare/regulations-guidance/electronic-prescribing>
+ <https://www.kaufmanhall.com/insights/thoughts-ken-kaufman/numbers-behind-national-hospital-flash-report>
+ <https://www.techtarget.com/searchhealthit/feature/How-4-EHR-Vendors-Are-Leveraging-Generative-AI-in-Clinical-Workflows>
+ <https://www.computerworld.com/article/3712789/how-rag-makes-generative-ai-tools-even-better.html>