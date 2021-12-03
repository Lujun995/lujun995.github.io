---
layout: post
title: Disentangling Key Indicators for Patients' Mortality Rate in Intensive Care Units
subtitle: By Team ABC (Ashley Hu, Betty Wu, and LuCas Zhang)
cover-img: /assets/img/ICU.jpg
thumbnail-img: /assets/img/ICU.jpg
share-img: /assets/img/ICU.jpg
tags: [Final Project]
---

*By Team ABC (**A**shley Hu, **B**etty Wu, and Lu**C**as Zhang)*

# 1. Introduction

Predicting patients’ in-hospital mortality is an area of active research. Accurate predictions of clinical outcomes enable clinicians to gauge patients' conditions and facilitate cost-effective management of hospital resources. Given the importance of this issue, in this project we are interested in **using a large-scale, high-resolution MIMIC-III dataset to predict in-hospital mortality**. Cleaning EHR data and defining the variables of interest as the first step lay the foundation of successful modeling in our project. **Particularly, I participated in the final project largely through data cleaning and creating documentation.** Here I focus on sharing my thoughts on data cleaning.

Please also visit our [GitHub repository](https://github.com/JiamanBettyWu/bios823_project) for accessing the source code.

# 2. Data Description

### 2.1. Data collection

The dataset MIMIC III (Medical Information Mart for Intensive Care) is used in our project. MIMIC III is a single-center database concerning patient admissions to critical care units. While full description can be found from [physionet.org](https://physionet.org/content/mimiciii/1.4/), the database incorporated several data sources, including:

- critical care information systems:
    - Two different systems: Philips **CareVue** and iMDsoft **MetaVision** ICU. Most of the data from the two systems were merged except for fluid intake. The data that are not merged will be given a suffix to distinguish the data source, e.g. "*CV*" for CareVue and "*MV*" for MetaVision. These two systems provided clinical data including:
        - physiological measurements, e.g. heart rate, arterial blood pressure, or respiratory rate;
        - documented progress notes by care providers;
        - drip medications and fluid balances.
- hospital electronic health records:
    - patient demographics
    - in-hospital mortality
    - laboratory test results, including hematology, chemistry and microbiology results.
    - discharge summaries
    - reports of electrocardiogram and imaging studies.
    - *billing-related information* such as International Classification of Disease, 9th Edition (ICD-9) codes, Diagnosis Related Group (DRG) codes, and Current Procedural Terminology (CPT) codes.
- Social Security Administration Death Master File.
    - Out-of-hospital mortality dates

### 2.2. De-identification Processes

Deidentified using *structured data cleansing* and *date shifting*. 

1. Removal of patient name, telephone number, address. 
2. Dates were shifted into the future. intervals preserved. Time of day, day of the week, and approximate seasonality were conserved.
3. **Patients > 89 yrs appear with ages of over 300 yrs.**
4. Protected health information was removed, such as diagnostic reports and physician notes.

### 2.3. Data Description

- **A relational database** consisting of 26 tables linked by identifiers which usually have the suffix ‘ID’, e.g. 
    - SUBJECT_ID (a unique patient)
    - HADM_ID (a unique admission to the hospital), and 
    - ICUSTAY_ID (a unique admission to an intensive care unit)
- Five tables track **patient stays**: ADMISSIONS; PATIENTS; ICUSTAYS; SERVICES; and TRANSFERS
- Another five tables are **dictionaries** (prefixed with ‘D_’): D_CPT; D_ICD_DIAGNOSES; D_ICD_PROCEDURES; D_ITEMS; and D_LABITEMS. 
    - Dictionary tables: Definitions for identifiers. E.g. ITEMID in CHARTEVENTS is explained in D_ITEMS, which represents the concept measured.
- The remaining tables are associated with patient care, such as physiological measurements, caregiver observations, and billing information.
- **‘Events’ tables**: a series of charted events such as notes, laboratory tests, and fluid balance. e.g. the OUTPUTEVENTS table: all measurements related to output for a given patient, the LABEVENTS table: laboratory test results for a patient.


# 3. Variables of Interest (Variable Definition)

Several kinds of variables may be of particular interest for the model construction, including ID variable, output and input variables. The ID variables are used as the primary keys which contain unique values for identification and are used to link records across different data sources throughout the MIMIC database. The output variable should indicate the clinical outcomes of each patient after his or her hospitalization. We use a Boolean variable as the model output. The input variables are typically the variables that we consider may relate to the clinical outcome, such as demographic information (e.g. age), the duration in the ICU, laboratory test results, and whether the patient has entered the ICU before. A detailed definition can be found in the list below.

### 3.1. ID Variables
- SUBJECT_ID: ID variable obtained from *ADMISSIONS.csv*. One unique subject id was assigned to each patient.
- HADM_ID: ID variable obtained from *ADMISSIONS.csv*. **Primary key**. One unique Hospital ADMinistration ID (HADM_ID) was assigned to each hospitalization of one patient, while multiple HADM_ID may correspond to the same person.
- ADMITTIME: datetime variable obtained from *ADMISSIONS.csv*. This variable records the time when the hospitalization started.
- DISCHTIME: datetime variable obtained from *ADMISSIONS.csv*. This variable records the time when the hospitalization ended.

### 3.2. Output:
- HOSPITAL_EXPIRE_FLAG: Boolean variable obtained from *ADMISSIONS.csv*. This variable indicates whether the patient passed away during hospitalization.

### 3.3 Input:
- ETHNICITY: categorical variable obtained from *ADMISSIONS.csv*. **Demographic information**.
- DIAGNOSIS: string variable obtained from *ADMISSIONS.csv*.
- GENDER: categorical variable obtained from *PATIENTS.csv* via joining on SUBJECT_ID. **Demographic information**. 
- DOB: datetime variable obtained from *PATIENTS.csv* via joining on SUBJECT_ID. **Demographic information**. 
- AGE_ON_AD: numeric variable derived from DOB and ADMITTIME. **Demographic information**. This variable represents the patients' age on administration. Age greater than 300 indicates patients' age greater than 89.
- SERVICES: multi-valued categorical variable obtained from *SERVICES.csv* via joining on SUBJECT_ID and HADM_ID. This variable indicates services that the patients utilized during hospitalization.
- ICU_STAY_DAYS: numeric variable derived from *ICUSTAY.csv*. This variable presents the total days of ICU stays during this hospitalization.
- MULTI_ENTRY_ICU: Boolean variable derived from *ICUSTAY.csv*. MULTI_ENTRY_ICU is set True if patients have entered the ICU before this hospitalization. 
- ICD9_code: ten Boolean variables derived from *PROCEDURES_ICD.csv* via joining on SUBJECT_ID and HADM_ID. These variables provided information on the ten most frequent procedures associated with hospitalization.
- TOTAL_ITEMID_code: twenty integer variables derived from *LABEVENTS.csv* via joining on SUBJECT_ID and HADM_ID. These variables provided information on the twenty most frequent lab tests during hospitalization. TOTAL_ITEMID_code indicates how many times the lab test has been carried out in each hospitalization.
- ABNORMAL_ITEMID_code: twenty integer variables derived from *LABEVENTS.csv* via joining on SUBJECT_ID and HADM_ID. Different from TOTAL_ITEMID_code, ABNORMAL_ITEMID_code indicates how many lab tests of this kind were abnormal.

### 3.4. ICD9 Code:

| ICD9 code | Explanation |
| :-: | :-: |
|3961|Extracorporeal circulat|
|3891|Arterial catheterization|
|3893|Venous cath NEC|
|8856|Coronar arteriogr-2 cath|
|9604|Insert endotracheal tube|
|966| Entral infus nutrit sub|
|9671|Cont inv mec ven <96 hrs|
|9672|Cont inv mec ven 96+ hrs|
|9904|Packed cell transfusion|
|9955|Vaccination NEC|

### 3.5. Lab Items code:

|ITEMID|LABEL|FLUID| CATEGORY|
| :-: | :-: | :-: | :-: |
|50820|pH|Blood|Blood Gas|
|50868|Anion Gap| Blood| Chemistry|
|50882|Bicarbonate| Blood| Chemistry|
|50902|Chloride| Blood| Chemistry|
|50912|Creatinine| Blood| Chemistry|
|50931|Glucose| Blood| Chemistry|
|50960|Magnesium| Blood| Chemistry|
|50970|Phosphate| Blood| Chemistry|
|50971|Potassium| Blood| Chemistry|
|50983|Sodium| Blood| Chemistry|
|51006|Urea Nitrogen| Blood| Chemistry|
|51221|Hematocrit| Blood| Hematology|
|51222|Hemoglobin| Blood| Hematology|
|51248|MCH| Blood| Hematology|
|51249|MCHC| Blood| Hematology|
|51250|MCV| Blood| Hematology|
|51265|Platelet Count| Blood| Hematology|
|51277|RDW | Blood| Hematology|
|51279|Red Blood Cells| Blood| Hematology|
|51301|White Blood Cells|Blood| Hematology|

# Solutions

In this section, we are going to generate these variables of interest from the MIMIC dataset using R. Note that the following code took a **considerable RAM (~20 GB)** and **several minutes** to be excuted using a single thread on a desktop computer.

{% highlight R %}
# load required packages
.libPaths("/admin/apps/rhel8/R-4.1.1/lib64/R/library")
library(tidyverse)
library(readr)
library(lubridate)

# load the MIMIC dataset, which can be accessed from https://physionet.org/content/mimiciii/1.4/
Admissions<- read_csv("/work/physionet.org/files/mimiciii/1.4/ADMISSIONS.csv.gz")
Service <- read_csv("/work/physionet.org/files/mimiciii/1.4/SERVICES.csv.gz")
Patients<- read_csv("/work/physionet.org/files/mimiciii/1.4/PATIENTS.csv.gz")
ICUstays <- read_csv("/work/physionet.org/files/mimiciii/1.4/ICUSTAYS.csv.gz")
lab_results<- read_csv("/work/physionet.org/files/mimiciii/1.4/LABEVENTS.csv.gz")
hospital_proc <- read_csv("/work/physionet.org/files/mimiciii/1.4/PROCEDURES_ICD.csv.gz")

# in the following code, we are going to generate the ICU_STAY_DAYS and the MULTI_ENTRY_ICU variables
# ICU_STAY_DAYS is the total days for which one patient stay in the ICU during one period of hospitalization
# MULTI_ENTRY_ICU is set to TRUE if the subject has entered ICU before this hospitalization according to our
#records. To accomplish this goal, we need a subquery to define the record for the first ICU entry.
ICUstays %>% group_by(SUBJECT_ID, HADM_ID) %>%
  summarise(ICU_STAY_DAYS = sum(LOS)) %>% 
  mutate(MULTI_ENTRY_ICU = !(HADM_ID %in% 
                              (ICUstays %>% 
                                 arrange(SUBJECT_ID, INTIME, ROW_ID) %>% 
                                 distinct(SUBJECT_ID, .keep_all = TRUE) %>%
                                 #this select hadm record when first entering ICU
                                 select(HADM_ID))$HADM_ID)) %>%
  arrange(SUBJECT_ID, HADM_ID) -> 
  ICUstays_df

# There are 164 unique icd9 codes, which can make the final table very sparse
# Most icd9 code may only occur few times, which can provide little information 
#in our model. There, we need to select the top X most frequent icd9 code. We also 
#need a subquery to search the most frequent icd9 code.
top = 10
hospital_proc %>% filter( ICD9_CODE %in%
                            (hospital_proc %>%
                               group_by(ICD9_CODE) %>%
                               summarise(COUNT = n()) %>%
                               slice_max(COUNT, n = top))$ICD9_CODE) %>%
  select(SUBJECT_ID, HADM_ID, ICD9_CODE) %>%
  group_by(SUBJECT_ID, HADM_ID, ICD9_CODE) %>%
  summarise(COUNT = n()) %>%
  pivot_wider(names_from = ICD9_CODE, values_from = COUNT,
              names_prefix = "ICD9_", values_fill = 0)->
  hospital_proc_df


#it appears that the lab_results will not have an hadm_id if 
#recorded before/after the hospitalization
#I may need to separate them into before, during, after tables
#here I only consider the results during the hospitalization
#it's hard to define what is the record prior to hospitalization
#Will a 3-day period work?

# As for the lab test results, we have the same problem with icd9 code because
#there are too many different kinds of lab tests. We also select the top 20 
#most frequent test.  
# In order to create one variable for each kind of lab test, we need to summarize
#how many abnormal tests exist during each hospitalization, compared with the total
#number of tests conducted.
top2 = 20
lab_results_during <- (lab_results %>% drop_na(HADM_ID) %>% 
                         mutate(flag = replace_na(FLAG, "normal")))
lab_results_during %>% filter(ITEMID %in% #top 20 frequent tests
                                (lab_results_during %>% group_by(ITEMID) %>%
                                   summarise(COUNT = n()) %>% 
                                   slice_max(COUNT, n = top2))$ITEMID) %>%
  group_by(SUBJECT_ID, HADM_ID, ITEMID) %>%
  summarise(TOTAL = n(), ABNORMAL = sum(FLAG %in% c("abnormal", "ABNORMAL", "Abnormal")), 
            PROP = ABNORMAL/TOTAL) %>%
  pivot_wider(names_from = ITEMID, values_from = c(TOTAL, ABNORMAL, PROP),
              names_prefix = "ITEMID_", values_fill = 0)->
  lab_results_during_df

# The following code is associated with the extraction of demographic information
Admissions %>% select(SUBJECT_ID, HADM_ID, ADMITTIME, DISCHTIME, 
                      ETHNICITY, DIAGNOSIS, HOSPITAL_EXPIRE_FLAG) ->
  admissions_df
Service %>% select(SUBJECT_ID, HADM_ID, CURR_SERVICE) %>%
  distinct(SUBJECT_ID, HADM_ID, CURR_SERVICE, .keep_all = TRUE) %>%
  group_by(SUBJECT_ID, HADM_ID) %>%
  mutate(SERVICES = paste0(CURR_SERVICE, collapse = ";")) %>%
  select(SUBJECT_ID, HADM_ID, SERVICES) %>%
  distinct(SUBJECT_ID, HADM_ID, .keep_all = TRUE)-> 
  service_df
Patients %>% select(SUBJECT_ID, GENDER, DOB) -> patients_df

# As the final step, we need to join these records (tables) on the primary 
#key (HADM_ID). We here use left (outer) join so that all records in 
#the admissions_df will be reserved regardless of whether the record is
#missing in other tables.
left_join(x= admissions_df, y= patients_df, 
          by = c("SUBJECT_ID")) -> temp
temp %>% mutate(AGE_ON_AD = 
                  as.numeric(as.duration(ADMITTIME - DOB), "year")) -> temp
left_join(x= temp, y = service_df, 
          by = c("SUBJECT_ID", "HADM_ID")) -> temp
left_join(x= temp, y = ICUstays_df,
          by = c("SUBJECT_ID", "HADM_ID")) -> temp
left_join(x= temp, y = hospital_proc_df,
          by = c("SUBJECT_ID", "HADM_ID")) -> temp
left_join(x= temp, y = lab_results_during_df,
          by = c("SUBJECT_ID", "HADM_ID")) -> temp
write_csv(temp, path = "~/MIMIC_cleaned.csv")
{% endhighlight %}

The final .csv file contains ~60,000 records and has a size of ~23 MB, compared to an original size of more than 50GB. Each record stands for one hospitalization indexed by a unique HADM_ID and has a detailed description of demographic information, ICU_stays information, lab test results, and hospitalization information (ICD9_code). This table is now ready for constructing a machine learning model.