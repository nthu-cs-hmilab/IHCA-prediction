# Multi-modal Prediciton of In-Hospital Mortality and In-Hospital Cardiac Arrest in the Intensive Care Unit

## Files tree
📂IHCA-prediction
├── 📂 model
│    ├──📂 .ipynb_checkpoints/
│    ├──📂 cardiac_validation/
│    ├──📂 cxr/
│    ├──📂 dead_validation/
│    ├──📂 early_fution_ca/
│    ├──📂 joint_fution_dead/
│    ├──📂 joint_fution_dead/
│    ├──📂 kears_model/
│    ├──📜 a test grad-cam_ca.ipynb
│    ├──📜 a tst grad-cam_dead.ipynb
│    ├──📜 a_graph.ipynb
│    ├──📜 copy_img.ipynb
│    ├──📜 copy_img_v2.ipynb
│    ├──📜 cut_hour.ipynb
│    ├──📜 cut_patients.ipynb
│    ├──📜 heatmap_mimic_ca.ipynb
│    ├──📜 heatmap_mimic_dead.ipynb
│    ├──📜 read_img.ipynb
│    ├──📜 test.ipynb
├─── 📂preprocess/
│    ├──📂 .ipynb_checkpoints/
│    ├──📂 ca
│    ├──📂 control
│    ├──📂 dead
│    └──📂 eicu
└──📜 README.md


## Introduction
In recent years, electronic health records (EHRs) have shown great promise in various clinical applications such as outcome prediction or information extraction. In this study, we developed several multi-modal machine learning approaches to predict in-hospital mortality and in-hospital cardiac arrest (IHCA) using EHR data. We used three types of data including time-independent (static) data (e.g., age, gender, and ethnicity), time dependent (dynamic) data (e.g., heart rate, respiratory rate, and peripheral oxygen saturation, and imaging data (chest X-ray (CXR) images). We hypothesized that using different types of patient data and combining them could help to better predict incident mortality and CA. We also validated the multi-modal in an independent cohort and compared the multi-modal to a previous scoring system.


## Methodology
- **Data**
    1. MIMIC-IV v1.0 Database:
        - Data on patients admitted to the ICU at Beth Israel Deaconess Medical Center (BIDMC).
        - Provides critical care de-identified data for over 40,000 patients.
        - patients are categorized based on their condition as follows: those identified with specific codes are recognized as having experienced a cardiac arrest, those marked as deceased in hospital records, excluding cardiac arrest cases, are classified as deceased, and all other patients are designated as part of the control group.
       
           ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/1094c929-38c9-42ce-860d-322b92ac9727)


    2. Electronic Intensive Care Unit Collaborative Research Database (eICU-CRD) v2.0:
        - Data on patients admitted to the ICU at 208 hospitals across the United States.
        - Provides critical care de-identified data for over 200,000 patients.
        - Patients diagnosed with specific ICD-9 codes are categorized as cardiac arrest patients. Those marked as 'Expired' in the patient table, excluding cardiac arrest cases, are classified as deceased; however, exact time of death is not utilized in the study. Patients not fitting into either cardiac arrest or deceased categories are considered normal patients. Only the first cardiac arrest event is considered if multiple events occurred during a patient's stay. This classification criterion is consistently applied to both the MIMIC-IV and eICU databases, streamlining the analysis process.
      
          ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/d7c35f5a-b760-4bb3-abb5-5936ea00c485)

  
    3. MIMIC-CXR Database:
        - Radiographic studies performed at Beth Israel Deaconess Medical Center in Boston.
        - CXR labels such as atelectasis, cardiomegaly, identified bythe subject id of patients
        - Includes 377,110 images corresponding to 227,835 radiographic studies.

---
  - **Extraction**
    1. Time-independent baseline features
       - Demographic information: includes gender, age, race, and type of intensive care unit admission
       - Chronic diseases: To select chronic diseases, such as metastatic cancer
       - Symptomatic illness: such as cardiac disease, according to the European Resuscitation Councill guidelines.
         
    2. Time series physiologic features:
       - physiological features included 6 VS, HR, RR, sBP, dBP, mean blood pressure(MBP), and perippheral oxygen saturation(SpO2) extracted at an hourly basis.
       - To address time series irregularity, multiple VSs within the same hour were combined, aggregating minimum, maximum, and average values hourly.(MIMIC-IV recorded VS hourly, whereas eICU recorded every five minutes.)
       - For the control group, the average of all 6 VSs was used, while for the incident group, specific conditions determined the use of minimum/maximum values for certain VSs.
       - Missing values were filled using the last observation carried forward method.
      
- **Model**
  - Multi-modal fuison : In this study, we compared three different types of data fusion flow and multi ple ML models to predict mortality and CA. Finally, we used the prediction results of CNN to optimize decision-level fusion
  - 
    1. Data-level fusion
       - we integrated the baseline features into time-series such that in every hour data contains VSs and baseline features. After that we used a LSTM model ca pable of learning from the temporal development of the features to predict events from 13 hours prior to the event.
         
         ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/49846ef2-f033-4fdf-8445-0146754f9549)
              
    2. Feature-level fusion
      - we extracted VS features by LSTM every hour throughout the 13 hours prediction period. We used the last LSTM layer output as static features add to baseline features. After that we used the following ML techniques that were evaluated in terms of prediction performance. The ML techniques consist of LR, XgBoost, RF, KNN, SVM.
    
        ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/d7251de2-88ae-4b82-b03e-4f343fcabd62)
 
    3. Decision-level fusion
       - we applied LSTM to predict events from 13 hours prior to the event by VSs. We applied ML techniques consisting of LR, XgBoost, RF, KNN and SVM to predict events by baseline features. After that we took the two outputs of the best static data prediction and the dynamic data prediction as the input. Finally, We applied five ML techniques as mentioned above to stack both the outcome from LR and LSTM to generate a new prediction.
      
         ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/475ed3be-77a7-4871-a7b5-0b72069447aa)

    4. Decision-level fusion combined with CNN model
       - After comparing three types of data fusion flow performance. Due to the great results of decision-level fusion along with the higher sensitivity were more suitable for our experimental subject. Therefore, we applied a CNN to predict mortality and CA through CXR images and we used the result from CNN to adjust the result from the stacked prediction by averaging the two predictions. The figure below shows the decision-level fusion was combined with a CNN model. Moreover, all experiments done in our study were implemented in Python with TensorFlow version 2.1.0 and Keras version 2.3.1. Our data preprocessing and model building were done with the introduction of the sklearn package version 0.23.2 and the imblearn package.
        
         ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/efcc3262-c594-4744-960a-41c5597c8fd1)

    
 
## Result
  - Trending of time-dependent(Collection of six VSs were based on time)
    - The figure below is  that in the MIMIC-IV data, the control group demonstrated a stable trend of all six VSs throughout the 24-hour collecting period. However, VSs of death and IHCA pa tients become progressive deterioration in the last several hours. It was noteworthy that the patients in the MIMIC-IV who developed CA or death exhibited, on av erage, more than 6 mmHg lower sBP, more than 1% lower SpO2 and more than 6 bpm higher resting HR compared to the control group.
      
      ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/eb042812-a232-422e-8c67-527b624b2597)

    - The figure below is that a similar VSs trajectory was seen in the eICU data. The patients in the eICU who developed CA exhibited on average, a 8 mmHg lower sBP, 1.5% lower SpO2 and a 3 bpm higher resting HR compared to the control group.
      
      ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/a19b9be2-e899-4832-9c76-fbe9aff04a48)

  - In our experiments we found the performance of three multi-modal fusions all had similar AUROC, but the fusion one was more unstable than others. The accuracy of fusion one of death was between 0.63 to 0.91 and the accuracy of fusion one of CA was between 0.6 to 0.92 from 13 hours prior to the event. The accuracy of fusion two of death was between 0.84 to 0.89 and CA was between 0.82 to 0.87 from 13 hours prior to the event. The accuracy of fusion three of death was between 0.89 to 0.86 and CA was between 0.81 to 0.83 from 13 hours prior to the event. The sensitivity of fusion two of death was between 0.78 to 0.85 and CA was between 0.72 to 0.78 was lower than fusion three. Therefore, we selected fusion three to be our study main target

      1. Data-level fusion
         - the AUC of prediction mortality reached the highest 0.95 and the AUC of prediction CA reached the highest 0.91 an hour before the event. But accuracy did not perform well at certain times. The accuracy of prediction mortality reached the lowest 0.63 and the accuracy of prediction CA reached the lowest 0.6 four hours before the event.

      2. Feature-level fusion
         - the LR, XGBoost and RF had better prediction results. The AUC of prediction mortality reached the highest 0.94 in the three models an hour before the event. The AUC of prediction CA among the three models the highest LR reaches 0.89, XGBoost 0.89 and RF 0.9. LR had the best sensitivity of prediction results. The sensitivity of prediction mortality above 0.78 and the sensitivity of prediction CA above 0.72 throughout the 13 hours prediction period.

      3. Decision-level fusion
         - All five models used to predict the stacking data had good pre diction results. The AUC of prediction mortality reached the highest 0.94 and the AUC of prediction CA reached the highest 0.91 that through LR and SVM an hour before the event. The overall performance was the best with SVM. For the entire 13-hours forecast period, the predicted mortality accuracy higher than 0.82, AUC higher than 0.9, sensitivity higher than 0.79 and specificity higher than 0.83; the predicted CA accuracy higher than 0.81, AUC higher than 0.88, sensi tivity higher than 0.77 and specificity higher than 0.82


  - Prediciton from the time-independent data
      - Patient demographics, comorbidities and presenting illness were first classified by LR. The discrimination of death of LR model with an AUROC: 0.85. Top five important features listed for death by LR include the ethnicity were unknown or other, ethnicity were unable to obtain, ethnicity were white, ethnicity were black or African American, and ethnicity were Asian. The discrimination of CA of LR model with an AUROC: 0.87. Top five important features listed for CA by LR include respiratory failure, spontaneous tension pneumothorax, age, renal failure, and acidosis. The figures below show the AUROC of five models for death and CA and the top ten important features. We also used these top ten features to train our model. We got the AUROC of the LR model as 0.76 and 0.84 for the prediction of death and CA, respectively. Experimental results show that using more features could help the predictions of the model.
    
        ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/0b1e597c-356a-420f-aefc-d41dc9121485)
    
        ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/26265882-a5d4-418c-87ef-c749137517d1)


  - Prediction from the time-dependent data
    - In the decision-level fusion we stacked the LR model with the LSTM model and combined both predictions from baseline features and VS features by using the SVM. In comparison to the baseline and VS only model, the stacked model exhibited consistently better predictions in death and CA. 
    
    ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/c0596aac-8e46-46f6-8333-51ae4ddd2c8c)
    ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/76bf2f92-ccc8-4855-bc77-dd84edd3753e)
    - We got the highest AUROC of 0.94 at one hour prior to the death and 0.91 at one hour prior to the CA as illustrated in above figures

    ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/9d3c0042-e964-498b-8e7f-633df20a4f9f)
    ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/897bc540-9286-45c7-b754-89de1e30c4b2)
    ![image](https://github.com/nthu-cs-hmilab/IHCA-prediction/assets/125988508/67262140-8230-48b2-952a-d813234ff84c)
    - Furthermore, we combined the fusion model and CNN model to get better performance. The results indicated that after combining the CNN model the sensitivity of the model was higher than before as illustrated in above Figure(Death sensitivity, CA sensitivity). In model calibration we got a steadily low Brier score between 0.11 to 0.13 throughout the 13 hours prediction time as illustrated in Figure(Birier score) .





## Conclusion
In this study, we developed an explainable multi-modal ML model for mortality and CA prediction in ICU from 13 hours prior to the event. Trained model and in ternally validated in MIMIC-IV and then externally validated in eICU. This model was based on static data, dynamic data and CXR images. During the development process, we compared the results of different fusion modality and different mod els adopted. The study showed that using the multi-modal can improve the final overall performance of the learning system. The fusion modality can provide new insight into complex interactions, the importance of explanatory variable trends, and non linearities. In the future, we will keep attempting other fusion modality and apply the method to other disease prediction tasks.




















