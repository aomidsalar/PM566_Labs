---
title: "Lab 7"
author: "Audrey Omidsalar"
date: "10/8/2021"
output:
  html_document:
    toc: yes
    toc_float: yes
    keep_md: yes
  github_document:
  always_allow_html: true
---



## Question 1: How many SARS-CoV2 papers?

```r
# Downloading the website
website <- read_html(x = "https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
numberpapers  <- stringr::str_extract(counts, "[:digit:]+.*[:digit:]")
```

#### There are 114,846 papers about SARS-CoV2 on PubMed

## Question 2: Academic publications on COVID19 and Hawaii


```r
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(
    db = "pubmed",
    term = "covid19 hawaii",
    retmax = 1000
  )
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

## Question 3: Get details about the articles

```r
#ids_list <- xml2::as_list(ids)
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[:digit:]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
#ids <- stringr::str_remove_all(ids, "<?/Id>")
```


```r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db = "pubmed",
    id = I(paste(ids, collapse = ",")),
    retmax = 1000,
    rettype = "abstract"
    )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

## Question 4: Distribution of universities, schools, and departments

```r
institution <- str_extract_all(
  str_to_lower(publications_txt),
  "university\\s+of\\s+(southern|new|northern|the|south|eastern|western)?\\s*[[:alpha:]-]+|[[:alpha:]-]+\\sinstitute\\s+of\\s+[[:alpha:]-]+"
  ) 
institution <- unlist(institution)
table(institution) %>% knitr::kable()
```



|institution                             | Freq|
|:---------------------------------------|----:|
|australian institute of tropical        |   13|
|beijing institute of pharmacology       |    2|
|berlin institute of health              |    4|
|broad institute of harvard              |    2|
|cancer institute of emory               |    2|
|cancer institute of new                 |    1|
|genome institute of singapore           |    1|
|graduate institute of rehabilitation    |    3|
|health institute of montpellier         |    1|
|i institute of marine                   |    1|
|leeds institute of rheumatic            |    2|
|massachusetts institute of technology   |    1|
|medanta institute of education          |    1|
|mediterranean institute of oceanography |    1|
|mgm institute of health                 |    1|
|monterrey institute of technology       |    1|
|national institute of allergy           |    1|
|national institute of environmental     |    3|
|national institute of public            |    1|
|national institute of technology        |    1|
|nordic institute of chiropractic        |    1|
|research institute of new               |    4|
|the institute of biomedical             |    1|
|the institute of medicine               |    1|
|university of alberta                   |    2|
|university of applied                   |    1|
|university of arizona                   |    5|
|university of arkansas                  |    1|
|university of basel                     |    8|
|university of benin                     |    1|
|university of botswana                  |    1|
|university of bradford                  |    1|
|university of bristol                   |    4|
|university of british                   |    4|
|university of calgary                   |    1|
|university of california                |   65|
|university of chicago                   |   11|
|university of cincinnati                |    9|
|university of colorado                  |    5|
|university of connecticut               |    1|
|university of copenhagen                |    1|
|university of córdoba                   |    1|
|university of education                 |    1|
|university of exeter                    |    1|
|university of florida                   |    5|
|university of granada                   |    2|
|university of haifa                     |    1|
|university of hawai                     |   92|
|university of hawaii                    |  180|
|university of hawaii-manoa              |    2|
|university of health                    |    8|
|university of hong                      |    1|
|university of honolulu                  |    3|
|university of illinois                  |    1|
|university of iowa                      |    4|
|university of jerusalem                 |    1|
|university of juiz                      |    4|
|university of kansas                    |    2|
|university of kentucky                  |    1|
|university of lausanne                  |    1|
|university of leeds                     |    2|
|university of louisville                |    1|
|university of malaya                    |    2|
|university of maryland                  |    9|
|university of medicine                  |    3|
|university of melbourne                 |    1|
|university of miami                     |    2|
|university of michigan                  |    8|
|university of minnesota                 |    4|
|university of murcia                    |    1|
|university of nebraska                  |    5|
|university of nevada                    |    1|
|university of new england               |    1|
|university of new south                 |    3|
|university of new york                  |    3|
|university of new york-university       |    1|
|university of north                     |    2|
|university of ontario                   |    1|
|university of oslo                      |    6|
|university of ottawa                    |    1|
|university of oxford                    |    9|
|university of paris                     |    1|
|university of pennsylvania              |   47|
|university of pittsburgh                |   13|
|university of porto                     |    2|
|university of puerto                    |    2|
|university of rio                       |    1|
|university of rochester                 |    4|
|university of sao                       |    2|
|university of science                   |   13|
|university of singapore                 |    1|
|university of south carolina            |    3|
|university of south florida             |    1|
|university of southern california       |   21|
|university of southern denmark          |    1|
|university of sydney                    |    1|
|university of technology                |    3|
|university of texas                     |    7|
|university of the health                |   16|
|university of the philippines           |    1|
|university of toronto                   |    5|
|university of toulon                    |    1|
|university of tübingen                  |    3|
|university of utah                      |    4|
|university of washington                |    6|
|university of wisconsin                 |    3|
|zoo-prophylactic institute of southern  |    2|

```r
schools_and_deps <- str_extract_all(
  str_to_lower(publications_txt),
  "school\\s+of\\s+[[:alpha:]-]+|department\\s+of\\s+[[:alpha:]-]+"
  )
table(schools_and_deps) %>% knitr::kable()
```



|schools_and_deps                  | Freq|
|:---------------------------------|----:|
|department of ageing              |    1|
|department of anatomy             |    2|
|department of anesthesia          |    2|
|department of anesthesilogy       |    1|
|department of anesthesiology      |    6|
|department of applied             |    3|
|department of biochemistry        |    1|
|department of biology             |   11|
|department of biosciences         |    1|
|department of biostatistics       |   15|
|department of botany              |    1|
|department of cardiology          |    1|
|department of cardiovascular      |    1|
|department of cell                |    4|
|department of chemistry           |    2|
|department of civil               |   12|
|department of clinical            |   10|
|department of commerce            |    1|
|department of communication       |    2|
|department of communicology       |    2|
|department of community           |    3|
|department of computational       |    1|
|department of critical            |    4|
|department of defense             |    1|
|department of dermatology         |   22|
|department of economics           |    3|
|department of education           |    7|
|department of emergency           |    5|
|department of environmental       |    6|
|department of epidemiology        |   18|
|department of experimental        |    1|
|department of family              |    6|
|department of general             |    3|
|department of genetic             |    1|
|department of geography           |    5|
|department of health              |   50|
|department of hematology          |    3|
|department of immunology          |    1|
|department of infectious          |   22|
|department of information         |    2|
|department of intensive           |    3|
|department of internal            |   55|
|department of international       |    1|
|department of kinesiology         |    2|
|department of laboratory          |    3|
|department of mathematics         |    7|
|department of mechanical          |    5|
|department of medical             |    7|
|department of medicine            |  110|
|department of microbiology        |    3|
|department of native              |    2|
|department of nephrology          |    5|
|department of neurological        |   12|
|department of neurology           |    2|
|department of neurosurgery        |    1|
|department of nursing             |    1|
|department of nutrition           |    7|
|department of ob                  |    5|
|department of obstetrics          |   18|
|department of occupational        |    2|
|department of orthopedic          |    5|
|department of otolaryngology-head |    4|
|department of paediatric          |    1|
|department of pathology           |   11|
|department of pediatric           |    2|
|department of pediatrics          |   26|
|department of pharmaceutical      |    1|
|department of pharmacology        |    2|
|department of pharmacy            |    1|
|department of physical            |    5|
|department of physiology          |   10|
|department of physiotherapy       |    1|
|department of population          |    6|
|department of preventive          |   13|
|department of psychiatry          |   27|
|department of psychology          |    7|
|department of public              |   10|
|department of pulmonary           |    1|
|department of quantitative        |    8|
|department of rehabilitation      |    6|
|department of research            |    1|
|department of rheumatology        |    7|
|department of smoking             |    8|
|department of social              |    1|
|department of sociology           |    4|
|department of sports              |    1|
|department of statistics          |    2|
|department of surgery             |   13|
|department of traffic             |    1|
|department of translational       |    1|
|department of tropical            |   31|
|department of twin                |    4|
|department of urology             |    1|
|department of veterans            |    8|
|department of veterinary          |    2|
|school of biomedical              |    3|
|school of brown                   |    2|
|school of education               |    2|
|school of electronic              |    1|
|school of epidemiology            |    6|
|school of health                  |    1|
|school of immunology              |    1|
|school of life                    |    1|
|school of medicine                |  343|
|school of natural                 |    1|
|school of nursing                 |   23|
|school of ocean                   |    1|
|school of pharmacy                |    1|
|school of physical                |    6|
|school of physiotherapy           |    1|
|school of population              |    2|
|school of public                  |   64|
|school of social                  |   11|
|school of transportation          |    1|

## Question 5: Form a Database

```r
##Extracting all Abstracts from html
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
abstracts <- str_extract(pub_char_list, "<Abstract>[[:print:][:space:]]+</Abstract>")
abstracts <- str_remove_all(abstracts, "</?[[:alnum:]- =\"]>+")
#replacing newlines or multiple spaces with single space
abstracts <- str_replace_all(abstracts, "[[:space:]]+", " ")
##checking how many abstracts are not provided
table(is.na(abstracts))
```

```
## 
## FALSE  TRUE 
##   114    37
```


```r
##Extracting all titles from html
titles <- str_extract(pub_char_list, "<ArticleTitle>[[:print:][:space:]]+</ArticleTitle>")
titles <- str_remove_all(titles, "</?[[:alnum:]-=\"]+>")
#checking how many titles there are
table(is.na(titles))
```

```
## 
## FALSE 
##   151
```
Putting everything into one dataframe `database`, and outputting first 10 rows into a table.

```r
database <- data.frame(
  PubMedId = ids,
  Title = titles,
  Abstract = abstracts
)
knitr::kable(database[1:10,], caption = "Some Papers on PubMed about SARS-CoV2 and Hawaii")
```



Table: Some Papers on PubMed about SARS-CoV2 and Hawaii

|PubMedId |Title                                                                                                                                                                    |Abstract                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|:--------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|34621978 |GenoRisk: A polygenic risk score for Alzheimer's disease.                                                                                                                |<Abstract> <AbstractText Label="Introduction" NlmCategory="UNASSIGNED">Recent clinical trials are considering inclusion of more than just apolipoprotein E (APOE) ε4 genotype as a way of reducing variability in analysis of outcomes.</AbstractText> <AbstractText Label="Methods" NlmCategory="UNASSIGNED">Case-control data were used to compare the capacity of age, sex, and 58 Alzheimer's disease (AD)-associated single nucleotide polymorphisms (SNPs) to predict AD status using several statistical models. Model performance was assessed with Brier scores and tenfold cross-validation. Genotype and sex × age estimates from the best performing model were combined with age and intercept estimates from the general population to develop a personalized genetic risk score, termed age, and sex-adjusted GenoRisk.</AbstractText> <AbstractText Label="Results" NlmCategory="UNASSIGNED">The elastic net model that included age, age x sex interaction, allelic APOE terms, and 29 additional SNPs performed the best. This model explained an additional 19% of the heritable risk compared to APOE genotype alone and achieved an area under the curve of 0.747.</AbstractText> <AbstractText Label="Discussion" NlmCategory="UNASSIGNED">GenoRisk could improve the risk assessment of individuals identified for prevention studies.</AbstractText> <CopyrightInformation>© 2021 The Authors. Alzheimer's &amp; Dementia: Diagnosis, Assessment &amp; Disease Monitoring published by Wiley Periodicals, LLC on behalf of Alzheimer's Association.</CopyrightInformation> </Abstract>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|34562997 |Variables Associated with Coronavirus Disease 2019 Vaccine Hesitancy Amongst Patients with Neurological Disorders.                                                       |<Abstract> <AbstractText Label="INTRODUCTION" NlmCategory="BACKGROUND">Given that the success of vaccines against coronavirus disease 2019 (COVID-19) relies on herd immunity, identifying patients at risk for vaccine hesitancy is imperative-particularly for those at high risk for severe COVID-19 (i.e., minorities and patients with neurological disorders).</AbstractText> <AbstractText Label="METHODS" NlmCategory="METHODS">Among patients from a large neuroscience institute in Hawaii, vaccine hesitancy was investigated in relation to over 30 sociodemographic variables and medical comorbidities, via a telephone quality improvement survey conducted between 23 January 2021 and 13 February 2021.</AbstractText> <AbstractText Label="RESULTS" NlmCategory="RESULTS">Vaccine willingness (n = 363) was 81.3%. Univariate analysis identified that the odds of vaccine acceptance reduced for patients who do not regard COVID-19 as a severe illness, are of younger age, have a lower Charlson Comorbidity Index, use illicit drugs, or carry Medicaid insurance. Multivariable logistic regression identified the best predictors of vaccine hesitancy to be: social media use to obtain COVID-19 information, concerns regarding vaccine safety, self-perception of a preexisting medical condition contraindicated with vaccination, not having received the annual influenza vaccine, having some high school education only, being a current smoker, and not having a prior cerebrovascular accident. Unique amongst males, a conservative political view strongly predicted vaccine hesitancy. Specifically for Asians, a higher body mass index, while for Native Hawaiians and other Pacific Islanders (NHPI), a positive depression screen, both reduced the odds of vaccine acceptance.</AbstractText> <AbstractText Label="CONCLUSION" NlmCategory="CONCLUSIONS">Upon identifying the variables associated with vaccine hesitancy amongst patients with neurological disorders, our clinic is now able to efficiently provide ancillary COVID-19 education to sub-populations at risk for vaccine hesitancy. While our results may be limited to the sub-population of patients with neurological disorders, the findings nonetheless provide valuable insight to understanding vaccine hesitancy.</AbstractText> </Abstract>                                                                                                                                              |
|34559481 |Astronomical Use of Nitrous Oxide Associated With Stress From the COVID-19 Pandemic and Lockdown.                                                                        |NA                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|34545941 |Cancer statistics for the US Hispanic/Latino population, 2021.                                                                                                           |<Abstract> <AbstractText>The Hispanic/Latino population is the second largest racial/ethnic group in the continental United States and Hawaii, accounting for 18% (60.6 million) of the total population. An additional 3 million Hispanic Americans live in Puerto Rico. Every 3 years, the American Cancer Society reports on cancer occurrence, risk factors, and screening for Hispanic individuals in the United States using the most recent population-based data. An estimated 176,600 new cancer cases and 46,500 cancer deaths will occur among Hispanic individuals in the continental United States and Hawaii in 2021. Compared to non-Hispanic Whites (NHWs), Hispanic men and women had 25%-30% lower incidence (2014-2018) and mortality (2015-2019) rates for all cancers combined and lower rates for the most common cancers, although this gap is diminishing. For example, the colorectal cancer (CRC) incidence rate ratio for Hispanic compared with NHW individuals narrowed from 0.75 (95% CI, 0.73-0.78) in 1995 to 0.91 (95% CI, 0.89-0.93) in 2018, reflecting delayed declines in CRC rates among Hispanic individuals in part because of slower uptake of screening. In contrast, Hispanic individuals have higher rates of infection-related cancers, including approximately two-fold higher incidence of liver and stomach cancer. Cervical cancer incidence is 32% higher among Hispanic women in the continental US and Hawaii and 78% higher among women in Puerto Rico compared to NHW women, yet is largely preventable through screening. Less access to care may be similarly reflected in the low prevalence of localized-stage breast cancer among Hispanic women, 59% versus 67% among NHW women. Evidence-based strategies for decreasing the cancer burden among the Hispanic population include the use of culturally appropriate lay health advisors and patient navigators and targeted, community-based intervention programs to facilitate access to screening and promote healthy behaviors. In addition, the impact of the COVID-19 pandemic on cancer trends and disparities in the Hispanic population should be closely monitored.</AbstractText> <CopyrightInformation>© 2021 The Authors. CA: A Cancer Journal for Clinicians published by Wiley Periodicals LLC on behalf of American Cancer Society.</CopyrightInformation> </Abstract>                                                                                                            |
|34536350 |Addendum needed on COVID-19 travel study.                                                                                                                                |NA                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|34532685 |Health Care Payers COVID-19 Impact Assessment: Lessons Learned and Compelling Needs.                                                                                     |NA                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|34529634 |Disaggregating Data to Measure Racial Disparities in COVID-19 Outcomes and Guide Community Response - Hawaii, March 1, 2020-February 28, 2021.                           |<Abstract> <AbstractText>Native Hawaiian and Pacific Islander populations have been disproportionately affected by COVID-19 (1-3). Native Hawaiian, Pacific Islander, and Asian populations vary in language; cultural practices; and social, economic, and environmental experiences,<sup>†</sup> which can affect health outcomes (4).<sup>§</sup> However, data from these populations are often aggregated in analyses. Although data aggregation is often used as an approach to increase sample size and statistical power when analyzing data from smaller population groups, it can limit the understanding of disparities among diverse Native Hawaiian, Pacific Islander, and Asian subpopulations<sup>¶</sup> (4-7). To assess disparities in COVID-19 outcomes among Native Hawaiian, Pacific Islander, and Asian populations, a disaggregated, descriptive analysis, informed by recommendations from these communities,** was performed using race data from 21,005 COVID-19 cases and 449 COVID-19-associated deaths reported to the Hawaii State Department of Health (HDOH) during March 1, 2020-February 28, 2021.<sup>††</sup> In Hawaii, COVID-19 incidence and mortality rates per 100,000 population were 1,477 and 32, respectively during this period. In analyses with race categories that were not mutually exclusive, including persons of one race alone or in combination with one or more races, Pacific Islander persons, who account for 5% of Hawaii's population, represented 22% of COVID-19 cases and deaths (COVID-19 incidence of 7,070 and mortality rate of 150). Native Hawaiian persons experienced an incidence of 1,181 and a mortality rate of 15. Among subcategories of Asian populations, the highest incidences were experienced by Filipino persons (1,247) and Vietnamese persons (1,200). Disaggregating Native Hawaiian, Pacific Islander, and Asian race data can aid in identifying racial disparities among specific subpopulations and highlights the importance of partnering with communities to develop culturally responsive outreach teams<sup>§§</sup> and tailored public health interventions and vaccination campaigns to more effectively address health disparities.</AbstractText> </Abstract>                                                                                                                                                                                                                                          |
|34499878 |Mass Critical Care Surge Response during COVID-19: Implementation of Contingency Strategies A Preliminary Report of findings from the Task Force for Mass Critical Care. |<Abstract> <AbstractText Label="BACKGROUND" NlmCategory="BACKGROUND">Following the publication of 2014 consensus statement regarding mass critical care during public health emergencies, much has been learned about surge responses and the care of overwhelming numbers of patients during the COVID-19 pandemic.<sup>1</sup> Gaps in prior pandemic planning were identified and require modification in the midst of ongoing surge throughout the world.</AbstractText> <AbstractText Label="METHODS" NlmCategory="METHODS">The Task Force for Mass Critical Care (TFMCC) adopted a modified version of established rapid guideline methodologies from the World Health Organization<sup>2</sup> and the Guidelines International Network-McMaster Guideline Development Checklist.<sup>3</sup> With a consensus development process incorporating expert opinion to define important questions and extract evidence, TFMCC developed relevant pandemic surge suggestions in a structured manner, incorporating peer-reviewed literature, "gray" evidence from lay media sources, and anecdotal experiential evidence.</AbstractText> <AbstractText Label="RESULTS" NlmCategory="RESULTS">Ten suggestions were identified regarding staffing, load-balancing, communication, and technology. Staffing models are suggested with resilience strategies to support critical care staff. Intensive care unit (ICU) surge strategies and strain indicators are suggested to enhance ICU prioritization tactics to maintain contingency level care and avoid crisis triage, with early transfer strategies to further load-balance care. We suggest intensivists and hospitalists be engaged with the incident command structure to ensure two-way communication, situational awareness, and the use of technology to support critical care delivery and families of patients in intensive care units (ICUs).</AbstractText> <AbstractText Label="CONCLUSIONS" NlmCategory="CONCLUSIONS">A subcommittee from the Task Force for Mass Critical Care offers interim evidence-informed operational strategies to assist hospitals and communities to plan for and respond to surge capacity demands from COVID-19.</AbstractText> <CopyrightInformation>Copyright © 2021. Published by Elsevier Inc.</CopyrightInformation> </Abstract>                                                                                                                                                                          |
|34491990 |Using test positivity and reported case rates to estimate state-level COVID-19 prevalence and seroprevalence in the United States.                                       |<Abstract> <AbstractText>Accurate estimates of infection prevalence and seroprevalence are essential for evaluating and informing public health responses and vaccination coverage needed to address the ongoing spread of COVID-19 in each United States (U.S.) state. However, reliable, timely data based on representative population sampling are unavailable, and reported case and test positivity rates are highly biased. A simple data-driven Bayesian semi-empirical modeling framework was developed and used to evaluate state-level prevalence and seroprevalence of COVID-19 using daily reported cases and test positivity ratios. The model was calibrated to and validated using published state-wide seroprevalence data, and further compared against two independent data-driven mathematical models. The prevalence of undiagnosed COVID-19 infections is found to be well-approximated by a geometrically weighted average of the positivity rate and the reported case rate. Our model accurately fits state-level seroprevalence data from across the U.S. Prevalence estimates of our semi-empirical model compare favorably to those from two data-driven epidemiological models. As of December 31, 2020, we estimate nation-wide a prevalence of 1.4% [Credible Interval (CrI): 1.0%-1.9%] and a seroprevalence of 13.2% [CrI: 12.3%-14.2%], with state-level prevalence ranging from 0.2% [CrI: 0.1%-0.3%] in Hawaii to 2.8% [CrI: 1.8%-4.1%] in Tennessee, and seroprevalence from 1.5% [CrI: 1.2%-2.0%] in Vermont to 23% [CrI: 20%-28%] in New York. Cumulatively, reported cases correspond to only one third of actual infections. The use of this simple and easy-to-communicate approach to estimating COVID-19 prevalence and seroprevalence will improve the ability to make public health decisions that effectively respond to the ongoing COVID-19 pandemic.</AbstractText> </Abstract>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|34481278 |The challenge of COVID-19 for adult men and women in the United States: disparities of psychological distress by gender and age.                                         |<Abstract> <AbstractText Label="OBJECTIVES" NlmCategory="OBJECTIVE">During the COVID-19 pandemic, the prevalence of psychological distress rose from 11% in 2019 to more than 40% in 2020. This study aims to examine the disparities among US adult men and women.</AbstractText> <AbstractText Label="STUDY DESIGN" NlmCategory="METHODS">We used 21 waves of cross-sectional data from the Household Pulse Survey that were collected between April and December 2020 for the study. The Household Pulse Survey was developed by the U.S. Census Bureau to document the social and economic impact of COVID-19.</AbstractText> <AbstractText Label="METHODS" NlmCategory="METHODS">The study population included four groups of adults: emerging adults (18-24 years); young adults (25-44 years); middle-aged adults (45-64 years); and older adults (65-88 years). Psychological distress was measured by their Generalized Anxiety Disorder score and the Patient Health Questionnaire. The prevalence of psychological stress was calculated using logistic models adjusted for socio-demographic variables including race/ethnicity, education, household income, and household structure. All descriptive and regression analysis considered survey weights.</AbstractText> <AbstractText Label="RESULTS" NlmCategory="RESULTS">Younger age groups experienced higher prevalence of psychological distress than older age groups. Among emerging adults, the prevalence of anxiety (42.6%) and depression (39.5%) was more than twice as high as older adults who experienced prevalence of anxiety at 20% and depression at 16.6%. Gender differences were also more apparent in emerging adults. Women between 18 and 24 years reported higher differential rates of anxiety and depression than those with men (anxiety: 43.9% vs. 28.3%; depression: 33.3% vs. 24.9%).</AbstractText> <AbstractText Label="CONCLUSION" NlmCategory="CONCLUSIONS">Understanding the complex dynamics between COVID-19 and psychological distress has emerged as a public health priority. Mitigating the negative mental health consequences associated with the COVID-19 pandemic, for younger generations and females in particular, will require local efforts to rebuild capacity for social integration and social connection.</AbstractText> <CopyrightInformation>Copyright © 2021 The Royal Society for Public Health. Published by Elsevier Ltd. All rights reserved.</CopyrightInformation> </Abstract> |


